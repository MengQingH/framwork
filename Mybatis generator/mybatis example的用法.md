在mybatis的generator配置文件中启用example，generator之后会自动生成对应的example文件，example文件中定义了很多字段的基本方法，如判断大小、判断为空等等，可以直接调用方法来实现sql查询，不用写sql。

* Example类：为每个表生成的sql封装类。包含一个oredCriteria属性，为Criteria类的列表，包含多个Criteria类，每个Criteria类为多个以and连接的条件，oredCriteria类以**or连接**这些条件。
* Criteria类：Example类中的内部类。继承了GeneratedCriteria类，只是继承了GeneratedCriteria类，没有添加其他的实现。类中包含一个Criterion类列表，包含多个Criterion类，每个Criterion类为一个sql条件，Criteria类中的这些条件**以and连接**。
* Criterion类：Example类中的内部类。该类为**一个sql条件**，会根据构造方法中传递的参数生成对应的条件。

简单来说：
```
Criterion：一个sql条件，如id is not null, id = 1
Criteria:包含多个sql条件，以and连接，比如：(id is not null) and (id > 1) and (id <10)
oredCriteria：包含多个and条件组合，这些条件组合之间以or连接，比如：(id is not null and id < 10) or (id is not null and id > 1)
```
```java
//Example:
protected List<Criteria> oredCriteria;

public void or(Criteria criteria) {
    oredCriteria.add(criteria);
}

public Criteria or() {
    Criteria criteria = createCriteriaInternal();
    oredCriteria.add(criteria);
    return criteria;
}

protected Criteria createCriteriaInternal() {
    Criteria criteria = new Criteria();
    return criteria;
}

//Criteria:
protected List<Criterion> criteria;
		
protected GeneratedCriteria() {
    super();
    criteria = new ArrayList<Criterion>();
}

protected void addCriterion(String condition, Object value, String property) {
    if (value == null) {
        throw new RuntimeException("Value for " + property + " cannot be null");
    }
    criteria.add(new Criterion(condition, value));
}

public Criteria andIdEqualTo(Integer value) {
    addCriterion("id =", value, "id");
    return (Criteria) this;
}

//Criterion:
```
使用方法：
1. 创建一个Example对象。
2. 使用or()方法并连续调用条件组合成多个and条件。
3. 如果要使用or条件，则多次调用or方法，创建多个Criteria对象。
4. 最后调用封装好的selectByExample方法，传入example对象。
```java
TaskExample taskExample = new TaskExample();
taskExample.or().andIdIsNotNull().andIdEqualTo(50);
taskExample.or().andIdIsNotNull().andIdEqualTo(51);
List<Task> tasks = taskMapper.selectByExample(taskExample);
```
原理：
1. example的无参or()方法会创建一个Criteria对象并放到oredCriteria列表中，并返回这个Criteria对象；
2. 调用返回的这个Criteria对象中封装好的条件方法，条件方法中传入对应的条件参数生成一个Criterion对象，并添加到Criterion列表中，