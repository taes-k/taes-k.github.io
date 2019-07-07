---
layout: default
comments: true
title: FilterChain&Filter
parent: java
date: 2019.07.07
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### Filter
필터는 HTTP 통신에서 Client의 Rquest와 Response를 목적에따라 변형시킬수 있는 역할을 합니다. 일반적으로 java.servlet.Filter에 선언되어있는 Filter를 상속받아 구현할수 있으며, '필터체인'이라는 주체에의해 필터들은 순차적으로 실행되게 됩니다.  

```c
public class SomeFilter implements Filter
{
    @Override
    public void init() throws ServletException
    {
        // TODO Auto-generated method stub
    }

    @Override
    public void destroy()
    {
        // TODO Auto-generated method stub
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException
    {
        chain.doFilter(httpsRequest, response);
    }

}
```  
  
위의 Filter를 상속받았을때 보이는것과 같이 doFilter에서는 3번째 인자로 FilterChain을 request, response와 함께 받아 Filter가 동작후, 다시 FilterChain의 doFilter를 실행해 다음 Filter를 실행시킬수 있도록 해줍니다.  

---  
  
### FilterChain
위에서 Filter를 실행해주는 Filterchain은 다음과 같이 구현해 줄 수 있습니다.  
```c
public class FilterChain {
    private ArrayList<Filter> filters;
    private Iterator<Filter> iteratorFilters;

    public ServletFilterChain(){}

    public void setFilters(ArrayList<Filter> filters) {
        this.filters = filters;
        this.iteratorFilters = filters.iterator();
    }

    public void doFilter(HttpRequest request, HttpResponse response) {
        if(iteratorFilters.hasNext()) {
            iteratorFilters.next().doFilter(request, response, this);
        }else {
            logger.info("Filter chain end");
        }
    }
}
```

위와같이 Filter list로 구현되어있는 필터들을 Filterchain에서 반복하여 수행해줌으로써 지정해준 필터들을 거쳐 request와 response들을 원하는대로 변형해 줄 수 있습니다.  

---   
  
### Wrapper class

Filter의 역할로써 Request와 Response를 변형 시켜준다고 말씁드렸습니다. 이때 사용되는게 Request, Response Wrapper class입니다. 보통 HttpServletRequestWrapper, HttpServletResponseWrapper를 상속받아 사용하게 되는데 필터를통해 변형을할때 Wrapper class를 선언하여 변경해주기 위해 사용 됩니다.

```c
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException
{
    RequestWrapper httpsRequestWrapper = new RequestWrapper((RequestWrapper)request);
    httpsRequestWrapper.something();
    chain.doFilter((ServletRequest)httpsRequestWrapper, response);
}

```

