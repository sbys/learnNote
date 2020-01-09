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
