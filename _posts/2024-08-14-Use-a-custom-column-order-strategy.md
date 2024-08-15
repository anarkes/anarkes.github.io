# Create tables with the columns in the same order as they were defined in the class

Hi! Today I was writing a REST API using **Spring Boot 3.3.2**, which will use an existing database schema, since I was working in local environment I decided to enable table creation feature using 
```properties
spring.jpa.hibernate.ddl-auto=create-drop
```
After some time of being testing I realized that the columns order were not created in the same order as they were defined in class, so the main task to create the api was out of my mind and my new goal was to create the columns in the same order as they were defined in class.

First of all I started looking into Google about this and I found that there was an easy way to custom how columns order was decided after **Hibernate 6.2** in this really good article [Ordering Columns in a table in JPA Hibernate](https://robertniestroj.hashnode.dev/ordering-columns-in-a-table-in-jpahibernate).

After read the article it was easier to write the following code to create table with columns in the same order as they were defined in class

```java
import org.hibernate.boot.Metadata;
import org.hibernate.boot.model.relational.ColumnOrderingStrategyLegacy;
import org.hibernate.cfg.AvailableSettings;
import org.hibernate.mapping.Column;
import org.hibernate.mapping.Table;

import org.springframework.boot.autoconfigure.orm.jpa.HibernatePropertiesCustomizer;
import org.springframework.context.annotation.Configuration;
import java.lang.reflect.Field;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

@Configuration
public class ClassDefinedColumnOrderingStrategy
				extends ColumnOrderingStrategyLegacy
				implements HibernatePropertiesCustomizer {

@Override
public List<Column> orderTableColumns(Table table, Metadata metadata){
	Class<?> mappedClass = metadata.getEntityBinding(metadata.getImports().get(table.getName())).getMappedClass();

	Map<String, Column> columnsMap = table
		.getColumns()
		.stream()
		.collect(Collectors.toMap(Column::getName, Function.identity()));

	return Arrays
		.stream(mappedClass.getDeclaredFields())
		.map(this::getFieldName)
		.map(columnsMap::get)
		.toList();
	}

	private String getFieldName(Field field) {
		String fieldName = null;
		if (field.isAnnotationPresent(jakarta.persistence.Column.class)) {
			jakarta.persistence.Column columnAnnotation = field.getAnnotation(jakarta.persistence.Column.class);
			fieldName = columnAnnotation.name();
		}

		return fieldName == null ? field.getName() : fieldName;
	}

	@Override
	public void customize(Map<String, Object> hibernateProperties) {
		hibernateProperties.put(AvailableSettings.COLUMN_ORDERING_STRATEGY,this);
	}
}
```

If you find a better way to achieve this please let me know.

