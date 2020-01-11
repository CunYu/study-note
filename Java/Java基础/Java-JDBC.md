### 概述

JDBC是Java程序用来和关系数据库交互的API，其全称为Java Data Base Connectivity。

### 示例

依赖

``` groovy
compile("mysql:mysql-connector-java:8.0.18")
```

``` java
Class.forName("com.mysql.jdbc.Driver");
Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306", "root", "root");
connection.setAutoCommit(false);

Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery("SELECT student_no, student_name FROM study.student WHERE student_grade = '1'");
while (resultSet.next()) {
    System.out.println(resultSet.getObject(1) + "-" + resultSet.getObject(2));
}
resultSet.close();
statement.close();

// 预编译
PreparedStatement preparedStatement = connection.prepareStatement("UPDATE study.student SET student_grade = ? WHERE student_no = ?");
preparedStatement.setInt(1, 2);
preparedStatement.setString(2, "2019001002");
int updateRow = preparedStatement.executeUpdate();
System.out.println(updateRow);

preparedStatement.close();
connection.close();
```

``` text
2019001001-小黑
2019001002-小白
1
```