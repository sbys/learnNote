# learnNote
使用
InputSteam is=Resources.getResourceAsStream(resouce);
SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(is);
SqlSession session=sqlSessionFactory.openSession();
#第一种
session.selectOne("com.example.dao.UserDao.getById");
#第二种
UserDao userDao=session.getMapper(UserDao.class);
userDao.getById(1);

对于spring 里面的SqlSessionTemplate只是持有sqlsessoin引用而已

Configuration，MapperRegistry负责读取，注入
对于各个Mapper，java动态代理生成代理类，invoke方法实现的逻辑由session执行实际的数据库操作

session执行操作：
  MapperStatement ms=configuration.getMappedStatement(statement);
  excutor.query(ms,,)
  handler.query(statment)
  handler前后可以执行过滤器
