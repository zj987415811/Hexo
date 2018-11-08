---
title: Druid源码解析之SQL解析
date: 2018-08-16 14:18:27
tags: Druid SQL解析
---

## Druid源码解析之SQL解析 ##
概述：
	SQL解析可以分为三层：
		- AST抽象语法树，语句解析-> MySqlStatementParser:多个表达式和词组成完整的语句 
			- SQLParser
				- errorEndPos:解析出错记录位置
				- lexer:词法解析
				- dbType:数据库类型
			- SQLStatement
				- exprParser:表达式解析类；
				- SQLCreateTableParser:建表语句解析类，其他DDL语句在SQLStatement中；
				- parseValueSize:记录解析结果集大小；
				- keepComments:是否保留注释；
				- parseCompleteValues:是否全部解析完成。
			- MySqlStatementParser
				- 静态关键词
				- exprParser:针对MySQL语句的parser;
		- 表达式解析-> MySqlExprParser:表达式由词组成，用来解析出不同表达式的含义
			- SQLExprParser 
				- AGGREGATE_FUNCTIONS 统计函数的关键词
				- aggregateFunctions 保存统计函数的关键词
			- MysqlExparser
				- AGGREGATE_FUNCTIONS 针对mysql统计函数的关键词

		- 词法解析-> MySqlLexer:解析出每个词的词义,部分关键词含义
			-   symbols_l2
				features
				text:保存目前整个SQL语句
				pos:当前处理位置
				mark:当前处理此开始位置
				ch:当前处理字符
				buf:当前缓存的处理词
				bufPos:用于取出词的标记，当前从text中取出的词应该是mark位置开始
				token:当前位于的关键词
				keywods:所有关键词集合
				stringVal
				lines:总行数
				dbType:数据库类型
	
	- SQL三个重要组成部分：Parser,AST,Visitor
		- Parser 由两部分组成
			- Lexer
			- 表达式
		- AST Parser的产物，语句经过词法分析，语法分析，其结构需要以一种计算机能读懂的方式表达出来，也就是抽象语法树。
		- Visitor 遍历这颗语法树，使用visitor模式，从根节点开始遍历，一直到最后一个叶子节点，在这个过程中，不断地收集信息到一个上下文中，整个遍历过程完成后，这棵树所表达的语法含义就被保存到上下文中。使用广度优先遍历方式。
					
1. 涉及到的类
	1. SQLStatementParser parser = new MySqlStatementParser(sql);
	2. MySqlStatementParser执行父类SQLExprParser的方法，这个过程在静态代码块中定义了一系列字段；
	static {
        String[] strings = { "AVG", "COUNT", "MAX", "MIN", "STDDEV", "SUM" };
       	AGGREGATE_FUNCTIONS_CODES = FnvHash.fnv1a_64_lower(strings, true);
        AGGREGATE_FUNCTIONS = new String[AGGREGATE_FUNCTIONS_CODES.length];
        for (String str : strings) {
            long hash = FnvHash.fnv1a_64_lower(str);
            int index = Arrays.binarySearch(AGGREGATE_FUNCTIONS_CODES, hash);
            AGGREGATE_FUNCTIONS[index] = str;
        }
    } 
	//执行MySqlExprParser的构造方法
	
	public MySqlExprParser(String sql){
        this(new MySqlLexer(sql));
        this.lexer.nextToken();
    } 

	//Lexer构造方法->SymbolTable初始化
	
	Map<String, Token> map = new HashMap<String, Token>();
	
	this.lexer.nextToken();

3.SQLStatement sqlStatement = parser.parseStatement();
	- parseStatement()中通过lexer.token选择对应的方法；
	- 如select->this.parseSelect();
	- 构造MySqlSelectParser对象
	MySqlSelectParser selectParser = createSQLSelectParser();
	SQLSelect select = selectParser.select();
	- select()方法
		- SQLSelect select = new SQLSelect();
		- 判断是否是with
		- 如果不是构造SQLSelectQuery
			- SQLSelectQuery query = query();query()方法内部
				- 判断token是否是lparen左括号;
				- 如果不是直接到MySqlSelectQueryBlock queryBlock = new MySqlSelectQueryBlock();并且设置父类的相关属性；
				- 如果token是select,判断查询缓存是否为空；如果不为空进行匹配；
					- selectListCache.match(laxer,queryBlock);
				- 如果token是select,执行laxer.nextTokenValue();执行下一个token解析；
				- Token token = lexer.token();得到下一个token后进行解析；
				- parseSelectList(SQLSelectQueryBlock queryBlock);
					- 构建selectList,final List<SQLSelectItem> selectList = queryBlock.getSelectList();
					- parseSelectItem();
					- expr = new SQLAllColumnExpr();
					- lexer.nextToken();
					- return new SQLSelectItem(expr, null, connectByRoot);
				- parseInto(queryBlock);
			- 将query设置到SQLSelect中;
			- SQLOrderBy orderBy = this.parseOrderBy();
4.MySqlSchemaStatVisitor visitor = new MySqlSchemaStatVisitor();
	- 父类构造方法
	 public SchemaStatVisitor(String dbType){
        this(new SchemaRepository(dbType), new ArrayList<Object>());
        this.dbType = dbType;
     }
	- SchemaRepository类作为构造函数入参传入，初始化SchemaRepository；
	- 初始化mysql相关：consoleVisitor = new MySqlConsoleSchemaVisitor();
	- 初始化MySqlConsoleSchemaVisitor的父类MySqlASTVisitorAdapter的父类SQLASTVisitorAdapter；
5.sqlStatement.accept(visitor)方法
	- priVisitor(this);这是一个空方法0-0
	- accept0(SQLASTVisitor visitor);
		- 执行实现类MySqlSchemaStateVisitor的visit(SQLSelectStatement x)；
			- repository.resolve(x);
				- 内部构造一个 SchemaResolveVisitor resolveVisitor = createResolveVisitor(options);
				- createResolveVisitor(SchemaResolveVisitor.Option ...);选择不同类型的数据库；
			- resolveVisitor.visit(stmt);
				- 内部执行 resolve(SchemaResolveVisitor visitor, SQLSelect x) 方法         
        		- 创建context环境：SchemaResolveVisitor.Context ctx = visitor.createContext(x);
        		- SQLSelectQuery query = x.getQuery();获取query;
        		- 之后执行accept(visitor)方法；
6.总结：新建MySqlStatementParser
	
	- 内部新建一个MySqlExprParser->MySqlLexer->读取第一个有效词:lexer.nextToken();
	- 新建一个MySqlLexer->执行父类Lexer构造方法->初始化关键词
	- new Lexer(String input, CommentHandler commentHandler) 或 new Lexer(String input, boolean skipComment)
		- input 输入语句
		- commentHandler 注释处理器
		- scanChar();读取第一个字符
	- 初始化Lexer后，回到MySqlExprParser构造器，初始化KeyWords集合;
	- 之后回到MySqlStatementParser:调用父类SQLStatementParser方法初始化;
	- new SQLParser(Lexer lexer, String dbType) 完成初始化。 
7.代码分析
		

	- `// 新建 MySQLParser
        SQLStatementParser parser = new MySqlStatementParser(sql);
        // 使用Parser解析生成AST，这里SQLStatement就是AST，解析结果是一个SQLStatement,是一个内部维护了树状逻辑结构的类
        SQLStatement statement = parser.parseStatement();

        // 使用visitor来访问AST
        MySqlSchemaStatVisitor visitor = new MySqlSchemaStatVisitor();
        statement.accept(visitor);
		`
	- 词法分析
		- SQLStatementParser parser = new MySqlStatementParser(sql);
		- 语法分析是SQLParser,词法分析Lexer,在Parser中拥有一个Lexer;
		- Lexer有两个，Lexer和其子类MySqlLexer;Lexer作为词法分析器，内部具有词汇表，以keywords表示;
		- keywords是以key为单词，value为Token的字典型结构，Token是单词的类型;
		- MySqlLexer类，除了父类的keywords外还有自己的keywords;Lexer中维护的是通用的，而MySqlLexer除了有通用还有MySQL数据库SQL方言关键字集合。
		- Parser是Lexer的使用者；
		- Lexer需要具备一个函数，能够让命令者命令它解析一个单词，而且Lexer还必须提供一个函数，供使用者获取Lexer上一次解析到的单词亿级单词的类型。
		- 在Lexer中，nextToken()方法提供了解析一个单词的需求，被调用时，就按顺序从SQL语句的开头到结尾，解析出下一个单词;
		- token()方法，则返回上一次解析的单词的Token类型，如果Token类型是标识符(Identifier)，Lexer还提供一个stringVal()方法，让使用者拿到标识符值。
		- nextToken()内部充斥着if语句和switch语句，解析单词时候，是一个个字符地解析，这个方法每次扫描一个字符，都必须判断单词是否结束，用什么方式来验证这个单词等待。这个过程是状态机运作的过程，每解析到一个字符，都要判断当前的状态，以决定应该进入下一个什么状态。
	- parseStatement()方法
		- if(lexer.token() == Token.XXX) return XXX;
		- 对应select类型的语句解析
			if (lexer.token() == Token.SELECT) {
    			statementList.add(parseSelect());
    			continue;
			}
		- parseSelect()方法
			public SQLStatement parseSelect() {
		        MySqlSelectParser selectParser = new MySqlSelectParser(this.exprParser);
		        
		        SQLSelect select = selectParser.select();
		        
		        if (selectParser.returningFlag) {
		            return selectParser.updateStmt;
		        }
		        
		        return new SQLSelectStatement(select, JdbcConstants.MYSQL);
 	    	}
			- 初始化一个针对MySQL Select语句的Parser,然后调用select()方法进行解析，把返回结果SQLSelect放到SQLSelectStatement里。而这个SQLSelectStatement就是AST抽象语法树，SQLSelect是他的第一个子节点。
     - MySqlSchemaStatVisitor visitor = new MySqlSchemaStatVisitor();
         - 有了AST语法树后，需要一个visitor来访问;
         - statement调用accept方法，以visitor作为参数，进行访问；statement的实际类型是SQLSelectStatement;
         - 在Druid中一条SQL语句中的元素，无论高层次还是低层次的元素，都是一个SQLObject,statement,expr,函数，字段，条件都是一种SQLObject。SQLObject是一个接口，accept方法由SQLObject定义，目的是为了让访问者在访问SQLObject时，告知访问者一些事情，在访问过程中手机关于SQLObject的一些信息。
         - accept()在SQLObjectImpl类实现
	         public final void accept(SQLASTVisitor visitor) {
			        if (visitor == null) {
			            throw new IllegalArgumentException();
			        }
			
			        visitor.preVisit(this);
					//真正执行的方法
			        accept0(visitor);
			
			        visitor.postVisit(this);
			 }
			 - 这是一个final方法，所有子类都要遵循这个模板，在accept()前后，visitor会进行一些操作，真正访问流程在acceptor0(visitor)中，这是一个抽象方法。
		 - acceptor0(SQLASTVisitor visitor)
			 protected void accept0(SQLASTVisitor visitor) {
        		if (visitor.visit(this)) {
            		acceptChild(visitor, this.select);
        		}
        		visitor.endVisit(this);
    	     }
			- 首先使用visitor访问自己，然后决定是否访问自己的子元素。
			- MySqlSchemaStateVisitor的visit()方法:初始化自己的aliasMap,之后返回true;
				public boolean visit(SQLSelectStatement x) {
        			setAliasMap();
        			return true;
    			}
			- 然后递归访问子元素
		- SQLObject负责通知visitor要访问的元素，visitor负责访问相应元素前中后三个过程的逻辑处理。
	

