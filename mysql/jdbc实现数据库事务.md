# JDBC实现数据库事务

数据库表：

```sql
CREATE TABLE `jdbc_student` (
  `id` int(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=17 DEFAULT CHARSET=utf8
```

实体类：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class JdbcStudent {
    private Integer id;
    private String name;
    private Integer age;
}
```

测试类：

```java
public class TransactionTest {
    public static void main(String[] args) throws Exception {
        TransactionTest test = new TransactionTest();
        Connection conn = test.getConn();
        // 先增加一条数据
       /* JdbcStudent js = new JdbcStudent();
        js.setId(1);
        js.setName("zhangsan");
        js.setAge(10);
         test.insert(conn,js);*/
        // test.testTransactionSuccess(conn);
        test.testTransactionRollBackSavePoint(conn);

    }
    // 测试事务成功
    public void testTransactionSuccess(Connection conn )throws Exception{
        conn.setAutoCommit(false);
        try {
            // 查询
            JdbcStudent student1 = queryById(conn, 1);
            // 更新
            student1.setName("lisi");
            update(conn,student1);
            // 插入
            JdbcStudent student2 = new JdbcStudent(2,"wangwu",20);
            insert(conn,student2);
            conn.commit();
        } catch (Exception e) {
            e.printStackTrace();
            conn.rollback();
        } finally {
            conn.close();
        }
    }

    // 测试失败事务 回滚到savePoint
    public void testTransactionRollBackSavePoint(Connection conn )throws Exception{
        conn.setAutoCommit(false);
        Savepoint savepoint = null;
        try {
            // 查询
            JdbcStudent student = queryById(conn, 1);
            // 更新
            student.setName("lisi");
            update(conn,student);
            savepoint = conn.setSavepoint();
            // 插入
            JdbcStudent student2 = new JdbcStudent(2,"wangwu",20);
            insert(conn,student2);
            // 故意抛出异常
            throwException();
            conn.commit();
        } catch (Exception e) {
            e.printStackTrace();
            if (savepoint != null){
                // 如果不为空回滚到插入lisi
                conn.rollback(savepoint);
                // 注意这里已经要提交
                conn.commit();
            }else {
                conn.rollback();
            }
        } finally {
            conn.close();
        }
    }

    // 测试失败事务全部回滚
    public void testTransactionFail(Connection conn )throws Exception{
        conn.setAutoCommit(false);
        try {
            // 查询
            JdbcStudent student = queryById(conn, 1);
            // 更新
            student.setName("lisi");
            update(conn,student);
            // 插入
            JdbcStudent student2 = new JdbcStudent(2,"wangwu",20);
            insert(conn,student2);
            // 故意抛出异常
            throwException();
            conn.commit();
        } catch (Exception e) {
            e.printStackTrace();
            conn.rollback();
        } finally {
            conn.close();
        }
    }
    public void throwException(){
        throw new RuntimeException("插入失败");
    }

    public void insert(Connection conn,JdbcStudent js) throws Exception {
        String sql = "insert into jdbc_student(id, name, age) values (?, ?, ?)";
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setInt(1,js.getId());
        ps.setString(2,js.getName());
        ps.setInt(3,js.getAge());
        int i = ps.executeUpdate();
        System.out.println("========= insert"+i+"  ============");
    }

    public JdbcStudent queryById(Connection conn,Integer id)throws Exception{
        String sql = "select * from jdbc_student where id = ?";
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setInt(1,id);
        ResultSet rs = ps.executeQuery();
        List<JdbcStudent> list = Lists.newArrayList();
        JdbcStudent js = null;
        while (rs.next()){
            js = new JdbcStudent(rs.getInt(1),rs.getString(2),rs.getInt(3));
            System.out.println(js);
            list.add(js);
        }
       return list.get(0);
    }

    public void update(Connection conn,JdbcStudent student)throws Exception{
        String sql = "update jdbc_student set name = ? where id = ?";
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setString(1,student.getName());
        ps.setInt(2,student.getId());
        int i = ps.executeUpdate();
        System.out.println("========== update "+i+"=======");
    }

    public Connection getConn()throws Exception{
        String url = "jdbc:mysql://10.30.0.249:3306/db_test?useUnicode=true&characterEncoding=utf-8";
        String userName = "dev";
        String password = "M20131209k";
        // 通过spi的方式加载mysql数据库驱动
        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = DriverManager.getConnection(url, userName, password);

        conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
        return conn;
    }
```

