---

comments: true
title: Spring AOP 切面编程介绍
sub_title: 
meta-keyword: spring boot,spring action,spring aop,AspectJ
meta-description: 本文简要地介绍了面向切面编程(AOP)的由来、使用场景，最后通过例子介绍了具体实现过程
categories: spring-boot
tags: spring-security
description: 本文以JdbcTemplate为例，介绍了通过使用模板代码去消除样板式代码的方法. 内容来源《Spring实战 第4版》
date: 2018-11-07
---

你是否写过这样的代码，当编写的时候总会感觉以前曾经这么写过。我的朋友，这不是似曾相识，这是样板式代码(boilerplate code)。
通常为了实现通用和简单的任务，你不得不一遍遍地重复编写这样的代码。

遗憾的是，它们中的很多都是因为使用Java API而导致的样板式代码。样板式代码的一个常见范例是使用JDBC访问数据库查询。举个例子，如果你曾经用过JDBC，那么你或许会写出类似下面的代码。

	public Employee getEmployeebyId(long id){
		Connection conn = null;
		PreparedStatement stmt = null;
		ResultSet rs = null;
		try{
			conn = dataSource.getConnection();
			stmt = conn.prepareStatement("select * from emplyoee where id=?");
			stmt.setLong(1,id);
			rs = stmt.executeQuery();
			Employee emplyoee = null;
			if(rs.next()){
				emplyoee = new Employee();
				emplyoee.setId(rs.getLong("id"));
				emplyoee.setFirstName(rs.getString("firstName"));
				emplyoee.setLastName(rs.getString("lastName"));
				emplyoee.setSalary(rs.getBigDecimal("salary"));
			}
			return emplyoee;			
		}catch(SQLException e){
			;
		}finally{
			if(rs != null){
				try{
					rs.close();
				}catch(SQLException e){}
			}
			if(stmt != null){
				try{
					stmt.close();
				}catch(SQLException e){}
			}
			
			if(conn != null){
				try{
					conn.close();
				}catch(SQLException e){}
			}
		}
		
	}
	
正如你所看到的，这段JDBC代码查询数据库获得员工姓名和薪水。从以上代码可以看到，少量查询员工的代码淹没在
一堆JDBC的样式代码中。首先，你要创建一个数据库连接，在创建一个语句对象，最后才能查询。中间还得捕捉SQLException,
这是一个检查型异常，即使它抛出后，你也做不了太多事情。

最后，该说的也说了，该做的也做了，你还不得不清理战场，关闭数据库连接、语句和结果集。同样地，你还是需要捕捉异常。

相同的，只有少量代码与查询员工逻辑有关系，其他代码都是JDBC样板代码。

JDBC不是产生样板式代码的唯一场景，在许多编程场景中往往都会导致类似的样板式代码，
JMS，JNDI和使用REST服务通常也涉及大量重复代码。

Spring旨在通过模板封装来消除样板式代码。Spring的JdbcTemplate使得执行数据库操作时，避免传统的JDBC样板式代码成为了可能

	
	public Employee getEmployeebyId(long id){
		return jdbcTemplate.queryForObject("select * from emplyoee where id=?",
			new RowMapper<Employee>(){
				public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
					Employee emplyoee = new Employee();
					emplyoee.setId(rs.getLong("id"));
					emplyoee.setFirstName(rs.getString("firstName"));
					emplyoee.setLastName(rs.getString("lastName"));
					emplyoee.setSalary(rs.getBigDecimal("salary"));
				return emplyoee;
				}
			},id);
	}
	
正如你所看到，新版本的 getEmployeebyId()简单多了。而且仅仅关注于从数据库中查询员工，模板的QueryForObject()
方法需要一个SQL查询语句和一个RowMapper对象，零个或多个查询参数。getEmployeebyId()方法再也看不到以前的JDBC样板式代码了，他们全部封装到了模板中。

下面看一下JdbcTemplate是如何封装的


在执行JdbcTemplate之前，先定义一个RowMapper


	RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<Employee>(Employee.class);
	
执行查询的语句变得更简单了，


	public Employee getEmployeebyId(long id){
		String sql="select * from emplyoee where id=?";
		return jdbcTemplate.queryForObject(sql,rowMapper,new Object[]{id});
	}

下面看一下queryForObject是如何实现的

	public <T> T queryForObject(String sql, RowMapper<T> rowMapper, Object... args) throws DataAccessException {
		List<T> results = query(sql, args, new RowMapperResultSetExtractor<T>(rowMapper, 1));
		return DataAccessUtils.requiredSingleResult(results);
	}

跳过中间调用的各种query，直接最终query的实现
	
	
	public <T> T query(
			PreparedStatementCreator psc, final PreparedStatementSetter pss, final ResultSetExtractor<T> rse)
			throws DataAccessException {

		Assert.notNull(rse, "ResultSetExtractor must not be null");
		logger.debug("Executing prepared SQL query");

		return execute(psc, new PreparedStatementCallback<T>() {
			public T doInPreparedStatement(PreparedStatement ps) throws SQLException {
				ResultSet rs = null;
				try {
					if (pss != null) {
						pss.setValues(ps);
					}
					rs = ps.executeQuery();
					ResultSet rsToUse = rs;
					if (nativeJdbcExtractor != null) {
						rsToUse = nativeJdbcExtractor.getNativeResultSet(rs);
					}
					return rse.extractData(rsToUse);
				}
				finally {
					JdbcUtils.closeResultSet(rs);
					if (pss instanceof ParameterDisposer) {
						((ParameterDisposer) pss).cleanupParameters();
					}
				}
			}
		});
	}
	
jdbc的样板代码就在这里实现了。

	
	public <T> T execute(PreparedStatementCreator psc, PreparedStatementCallback<T> action)
			throws DataAccessException {

		Assert.notNull(psc, "PreparedStatementCreator must not be null");
		Assert.notNull(action, "Callback object must not be null");
		if (logger.isDebugEnabled()) {
			String sql = getSql(psc);
			logger.debug("Executing prepared SQL statement" + (sql != null ? " [" + sql + "]" : ""));
		}


		Connection con = DataSourceUtils.getConnection(getDataSource()); 
		PreparedStatement ps = null;
		try {
			Connection conToUse = con;
			if (this.nativeJdbcExtractor != null &&
					this.nativeJdbcExtractor.isNativeConnectionNecessaryForNativePreparedStatements()) {
				conToUse = this.nativeJdbcExtractor.getNativeConnection(con);
			}
			ps = psc.createPreparedStatement(conToUse);
			applyStatementSettings(ps);
			PreparedStatement psToUse = ps;
			if (this.nativeJdbcExtractor != null) {
				psToUse = this.nativeJdbcExtractor.getNativePreparedStatement(ps);
			}
			T result = action.doInPreparedStatement(psToUse);
			handleWarnings(ps);
			return result;
		}
		catch (SQLException ex) {
			// Release Connection early, to avoid potential connection pool deadlock
			// in the case when the exception translator hasn't been initialized yet.
			if (psc instanceof ParameterDisposer) {
				((ParameterDisposer) psc).cleanupParameters();
			}
			String sql = getSql(psc);
			psc = null;
			JdbcUtils.closeStatement(ps);
			ps = null;
			DataSourceUtils.releaseConnection(con, getDataSource());
			con = null;
			throw getExceptionTranslator().translate("PreparedStatementCallback", sql, ex);
		}
		finally {
			if (psc instanceof ParameterDisposer) {
				((ParameterDisposer) psc).cleanupParameters();
			}
			JdbcUtils.closeStatement(ps);
			DataSourceUtils.releaseConnection(con, getDataSource());
		}
	}