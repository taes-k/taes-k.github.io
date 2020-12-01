---
layout: post
comments: true
title: java.nio.files.Files.lines에 대한 탐구
tags: [java, nio, file]
---

### OOM이 터질수 있는 File read 인가

File에 존재하는 line count 를 구하기 위해 아래 메서드를 사용해 보았습니다.  

```java
try(Stream<String> lines = Files.lines(path))
{
  count = lines.count();
}  
```

위 코드를 PR올린 후 팀원분께서 '혹시 사이즈가 큰 파일을 받게되면 OOM이 터지지 수 있을까요?' 라는 리뷰를 남겨주셨습니다.

--- 

### 관련문서

> Read all lines from a file as a Stream. Unlike readAllLines, this method does not read all lines into a List, but instead populates lazily as the stream is consumed.
> 
> Bytes from the file are decoded into characters using the specified charset and the same line terminators as specified by readAllLines are supported.
> 
> The returned stream contains a reference to an open file. The file is closed by closing the stream.
> 
> The file contents should not be modified during the execution of the terminal stream operation. Otherwise, the result of the terminal stream operation is undefined.
> 
> After this method returns, then any subsequent I/O exception that occurs while reading from the file or when a malformed or unmappable byte sequence is read, is wrapped in an UncheckedIOException that will be thrown from the Stream method that caused the read to take place. In case an IOException is thrown when closing the file, it is also wrapped as an UncheckedIOException.

본문 : https://docs.oracle.com/javase/9/docs/api/java/nio/file/Files.html#lines-java.nio.file.Path-java.nio.charset.Charset- 

해당 메서드를 설명하는 문서상으로는 `Files.readAllLines()`과는 다르게 모든 행을 List로 읽는 대신 `Lazy production Stream`을 통해 전체 파일을 읽는다고 명확하게 설명이 나와있습니다. 

---

### 실제 테스트

```java
public int totalProductCount()
{
    initializeCheck();
 
    int rowCount;
    try (Stream<String> fileStream = Files.lines(this.filePath))
    {
        rowCount = (int) fileStream.count() - 1;
    }
    catch (Exception e)
    {
        throw new EpException(ErrorCodes.EP_FILE_ERROR, e);
    }
 
    return rowCount;
}
```

위 의 코드를 통해 실제 메모리 사용량을 테스트 해보았습니다.

#### Test 1 → 500만건 (File 용량 : 3.82GB)
![1]({{ site.images | relative_url }}/posts/2020-12-01-java-nio-files-lines/1.png)   

- (테스트 과정중 파일 다운로드가 포함되어 있음)
- 메모리 사용량 100MB 이하로 빠르게 수행됨  

#### Test 2 → 5000만건 (File 용량 : 24.42GB)
![2]({{ site.images | relative_url }}/posts/2020-12-01-java-nio-files-lines/2.png) 

- (테스트 과정중 파일 다운로드가 포함되어 있음)
- 메모리 사용량 100MB 이하로 빠르게 수행됨  


#### 결론

- File stream 으로 제공되어 OOM에서 안전하게 File read 가능함
- 내부적으로 BufferReader를 통해 reader stream 열어서 제공됨

---

### 내부 코드 분석

위 제공받는 `Stream`은 실제로 비동기 로직에서 `Stream`을 제공받듯이 file을 read 하고 있습니다.  
순수 자바코드로 어떻게 File을 `Stream`으로 제공 할 수 있는지 내부 코드를 분석해보도록 하겠습니다.

```java
// java.nio.files.Files
 
public static Stream<String> lines(Path path, Charset cs) throws IOException {
    BufferedReader br = Files.newBufferedReader(path, cs);
    try {
        return br.lines().onClose(asUncheckedRunnable(br));
    } catch (Error|RuntimeException e) {
        try {
            br.close();
        } catch (IOException ex) {
            try {
                e.addSuppressed(ex);
            } catch (Throwable ignore) {}
        }
        throw e;
    }
}
```


```java
// java.io.BufferedReader
 
 
public Stream<String> lines() {
    Iterator<String> iter = new Iterator<String>() {
        String nextLine = null;
 
        @Override
        public boolean hasNext() {
            if (nextLine != null) {
                return true;
            } else {
                try {
                    nextLine = readLine();
                    return (nextLine != null);
                } catch (IOException e) {
                    throw new UncheckedIOException(e);
                }
            }
        }
 
        @Override
        public String next() {
            if (nextLine != null || hasNext()) {
                String line = nextLine;
                nextLine = null;
                return line;
            } else {
                throw new NoSuchElementException();
            }
        }
    };
    return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
            iter, Spliterator.ORDERED | Spliterator.NONNULL), false);
}
```


```java
// java.util.stream
 
 
public static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel) {
    Objects.requireNonNull(spliterator);
    return new ReferencePipeline.Head<>(spliterator, StreamOpFlag.fromCharacteristics(spliterator),  parallel);
}
 
```

```java
// Java.util.Spliterators
 
 
public static <T> Spliterator<T> spliteratorUnknownSize(Iterator<? extends T> iterator,
                                                        int characteristics) {
    return new IteratorSpliterator<>(Objects.requireNonNull(iterator), characteristics);
}
 
 
```

```java
/**
 * Source stage of a ReferencePipeline.
 *
 * @param <E_IN> type of elements in the upstream source
 * @param <E_OUT> type of elements in produced by this stage
 * @since 1.8
 */
static class Head<E_IN, E_OUT> extends ReferencePipeline<E_IN, E_OUT> {
...
}
 
 
abstract class ReferencePipeline<P_IN, P_OUT>
        extends AbstractPipeline<P_IN, P_OUT, Stream<P_OUT>>
        implements Stream<P_OUT> {
...
}
```