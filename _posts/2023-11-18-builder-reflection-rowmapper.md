---
layout: post
comments: true
title: Lombok builder reflection rowmapper 
tags: [lombok, builder, reflection, rowmapper]
---

### Lombok builder reflection rowmapper 

작업중 lombok builder를 바로 사용하지 못하고 reflection 을 통해 rowmapper를 구성해야 할 일이 생겨 작업했던 코드를 기록합니다.  

여러 타입들에 대해 사용될 수 있는 공통적인 rowMapper 구성 필요시 유용 할 수 있습니다.

```java
public class BuilderRowMapper<T, Builder> implements RowMapper<T> {

	private final CustomEncryptorDecryptor customEncryptorDecryptor;

	private final Class<T> tClazz;

	public OracleRowMapper(Class<T> tClazz, CustomEncryptorDecryptor customEncryptorDecryptor) {
		this.tClazz = tClazz;
		this.customEncryptorDecryptor = customEncryptorDecryptor;
	}

	@SuppressWarnings("unchecked")
	@SneakyThrows
	@Override
	public T mapRow(ResultSet rs, int rowNum) {
		Builder builder = (Builder)tClazz.getDeclaredMethod("builder").invoke(null);

		Field[] fields = tClazz.getDeclaredFields();
		for (Field field : fields) {
			if (!field.isAnnotationPresent(Column.class) || field.isAnnotationPresent(ReadOnlyProperty.class)) {
				continue;
			}

			Method[] superBuilderMethods = builder.getClass().getSuperclass().getDeclaredMethods();
			setBuilderProperty(rs, field, superBuilderMethods, builder);
		}

		Field[] superFields = tClazz.getSuperclass().getDeclaredFields();
		for (Field field : superFields) {
			if (!field.isAnnotationPresent(Column.class) || field.isAnnotationPresent(ReadOnlyProperty.class)) {
				continue;
			}

			Method[] superSuperBuilderMethods =
				builder.getClass().getSuperclass().getSuperclass().getDeclaredMethods();
			setBuilderProperty(rs, field, superSuperBuilderMethods, builder);
		}

		return (T)builder.getClass().getSuperclass().getDeclaredMethod("build").invoke(builder);
	}

	private <T> void setBuilderProperty(
		ResultSet rs,
		Field field,
		Method[] superBuilderMethods,
		T builder
	) throws SQLException, InvocationTargetException, IllegalAccessException {
		for (Method builderMethod : superBuilderMethods) {
			if (!builderMethod.getName().equals(field.getName())) {
				continue;
			}

			Object rowValue = rs.getObject(field.getDeclaredAnnotation(Column.class).value());

			if (rowValue == null) {
				builderMethod.invoke(builder, rowValue);
				continue;
			}

			if (Integer.class.isAssignableFrom(field.getType())
				|| int.class.isAssignableFrom(field.getType())) {
				builderMethod.invoke(builder, Integer.parseInt(rowValue.toString()));
			} else if (Long.class.isAssignableFrom(field.getType())
				|| long.class.isAssignableFrom(field.getType())) {
				builderMethod.invoke(builder, Long.parseLong(rowValue.toString()));
			} else if (String.class.isAssignableFrom(field.getType())) {
				builderMethod.invoke(builder, rowValue.toString());
			} else if (Boolean.class.isAssignableFrom(field.getType())
				|| boolean.class.isAssignableFrom(field.getType())) {
				builderMethod.invoke(builder,
					"1".equals(rowValue.toString())
						|| "TRUE".equalsIgnoreCase(rowValue.toString())
				);
			} else if (Instant.class.isAssignableFrom(field.getType())) {
				Instant instant = Times.fromExtendedSimpleFormat(rowValue.toString());
				builderMethod.invoke(builder, instant);
			} else if (CustomEnum.class.isAssignableFrom(field.getType())) {
				CustomEnum customEnum = getCustomEnum(field, rowValue);
				builderMethod.invoke(builder, etcCodeEnum);
			} else if (Encrypted.class.isAssignableFrom(field.getType())) {
				builderMethod.invoke(builder,
					Encrypted.from(customEncryptorDecryptor.decrypt(rowValue.toString()))
				);
			} else {
				builderMethod.invoke(builder, rowValue);
			}
		}
	}

	private CustomEnum getCustomEnum(Field field, Object rowValue) {
		return Arrays.stream(field.getType().getEnumConstants())
			.map(it -> (CustomEnum)it)
			.filter(it -> Objects.equals(it.getCustom(), rowValue))
			.findFirst()
			.orElseThrow(
				() -> new FailedConversionException(
					"No enum constant " + field.getType().getName() + "." + rowValue)
			);
	}
}


```