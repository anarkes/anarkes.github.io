## Use a custom ColumnOrderStrategy with Spring Boot

Today I was creating a REST API using spring boot, and for testing I enabled the automatic
creation of tables using spring.jpa.hibernate.ddl-auto=create-drop and I realized that
the columns are created in a different way than how those where defined in the class. I decided
to write a code to create tables using the same field order in class.

Here you could find the code I wrote to achieve this
https://gist.github.com/anarkes/07e831eb2dea8c42e73fa3dcfb3ad37b
