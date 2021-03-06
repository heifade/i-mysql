i-mysql
=======


# 安装
```bash
npm install i-mysql
```
# 介绍


##i-mysql的主要特点：
* 1.多数据库自由切换。
* 2.数据库托管。
* 3.简单的数据库执行方法封装。
* 4.单表CRUD封装。
* 5.事务封装（超时自动提交、错误自动回滚）。
* 6.支持连缀写法。


##方法总览：
> i-mysql
>
> > config
>
> > defaultDb
>
> > db
>
> > > getDbIndex
>
> > > switch
>
> > > sql
>
> > table
>
> > > getTableName
>
> > > getDbIndex
>
> > > switch
>
> > > insert
>
> > > select
>
> > > update
>
> > > delete
>
> > transaction
>
> > > getId
>
> > > getDbIndex
>
> > > autoCommit
>
> > > destroy
>
> > > switch
>
> > > commit
>
> > > rollback
>
> > > table
>
> > > > insert
>
> > > > select
>
> > > > update
>
> > > > delete
>
> > > sql


##一、数据库配置
i-mysql必须在执行config之后才可以进行其它操作，且只能config一次。<br />
function config(configs) config方法的configs参数可以为空（为空时config方法会返回上次调用config方法成功时的configs参数），也可接收如下例子中的数组，当然也可以是只有单个数据库配置的json对象（这个时候就不存在数据库切换的情况了）。<br />

```js
var iMysql = require('i-mysql');
//配置
iMysql.config([
    {host:'127.0.0.1',port:3306,database:'db0',user:'root',password:'123456'},
    {host:'127.0.0.1',port:3306,database:'db1',user:'root',password:'123456'}
]);

//获取配置(可用来判断是否已经配置过i-mysql)
console.log(iMysql.config());
```


##二、数据库索引
成功配置了i-mysql之后，会默认将第一个数据库作为默认数据库，您也可以通过defaultDb方法获取当前默认数据库的索引或者设置新的默认数据库索引给i-mysql。<br />
数据库索引值从0开始，即为config时数组的索引，如果只有单数据库，那么有效的数据库索引值只能为0。<br />
获取或设置数据库索引的方法，function defaultDb(dbIndex) defaultDb方法支持数字型参数或者一个返回数字型数据的function参数。<br />

获取默认数据库的索引：
```js
var defaultDbIndex = iMysql.defaultDb();
```

设置新的默认数据库索引：
```js
iMysql.defaultDb(1);
```

动态指定默认的数据库索引：
```js
function chooseDb(){
    return 1;
}

iMysql.defaultDb(chooseDb);
```

##三、简单的数据库执行对象
* 1)获取简单数据库执行对象，function db(dbIndex) db方法支持数字型参数或者一个返回数字型数据的function参数。
* 2)获取简单数据库执行对象所在数据库索引，function db.getDbIndex() getDbIndex方法返回当下简单数据库对象所在的数据库索引。
* 3)数据库执行对象切换数据库，function db.switch(dbIndex) switch方法支持数字型参数或者一个返回数字型数据的function参数。
* 4)简单数据库执行对象执行sql，function db.sql(sql,options,cb) sql方法可接受3个参数（与mysql包的query方法相同）：
  * ①必填参数sql：可以是字符串或者一个返回字符串数据的function。
  * ②可选参数options：用来填充sql的对象。
  * ③可选参数cb：执行sql之后的回调函数。

###1.简单数据库对象的获得途径

获得一个简单数据库执行对象：
```js
var defaultDbObj = iMysql.db();//db方法没有传入时，i-mysql会根据默认的数据库获得一个简单数据库执行对象
```

指定从某个数据库中获得一个简单数据库执行对象：
```js
var db1Obj = iMysql.db(1);
```

动态指定从某个数据库中获得一个简单数据库执行对象：
```js
function chooseDb(){
    return 1;
}

var dbObj = iMysql.db(chooseDb);
```

###2.简单数据库对象获取当下所在的数据库索引

```js
var db1Obj = iMysql.db(1);
console.log(db1Obj.getDbIndex());
```

###3.简单数据库对象切换数据库

```js
var db1Obj = iMysql.db(1);
db1Obj.switch(0);
```

###4.简单数据库对象执行sql

```js
var db1Obj = iMysql.db(1);
db1Obj.sql('select c1 from test_table where id = ?','3',function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###5.简单数据库对象连缀写法

```js
var db1Obj = iMysql.db(1);
db1Obj.sql('select c1 from test_table where id = ?','3',function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
}).sql('select c1 from test_table where ?',{id:'4'},function(err){//第二个参数如果是json对象，比如{id:'4',name:'t1'}，那么在执行时会格式化成`id`='4',`name`='t1'
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
}).switch(0).sql('select c1 from test_table where id = 5',function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```


##四、table对象
* 1)获取table对象，function table(dbIndex,tableName) table支持2个参数，dbIndex可选，tableName必填，如果只有1个参数传入则被认为该参数表示的是tableName，其中dbIndex支持数字型参数或者一个返回数字型数据的function参数，没有传入dbIndex时默认为i-mysql的默认数据库索引，tableName可以是表示表名的字符串或者一个可以返回表名字符串数据的方法。
* 2)从table对象获取表名，function table.getTableName()。
* 3)从table对象获取所在数据库索引，function table.getDbIndex()。
* 4)table对象切换数据库，function table.switch(dbIndex) switch方法支持数字型参数或者一个返回数字型数据的function参数。
* 5)table对象insert数据，function table.insert(options,cb) insert方法支持2个参数：
  * ①可选参数options：{data:...,fields:...,values:...,freestyleFields:...}，data优先与field和values，data为如{'字段':字段值}的json对象，fields和values配套使用，其中fields表示字段(可支持如['字段1']的数组或者以逗号间隔的字符串)，values表示对应的值(可支持如['字段值','字段值']的数组或者[['字段值','字段值1'],['字段值','字段值2']]这样的二维数组从而进行批量插入)，freestyleFields表示那些字段对应的值不需要做合法性校验和escape的自由字段(可支持如['字段1']的数组或者以逗号间隔的字符串)，其作用在data和values中有效。
  * ②可选参数cb：执行sql之后的回调函数，该回调函数的参数同mysql包的query方法中cb的参数。
* 6)table对象select数据，function table.select(options,cb) select方法支持2个参数：
  * ①可选参数options：{fieldset:...,fields:...,where:...,groupBy:...,orderBy:...,limit:...,freestyleFields:...}，其中fieldset可以为字符串，fields表示字段(可支持如['字段1']的数组或者以逗号间隔的字符串，为空时默认查询所有字段)，where支持如{'字段1':字段值,'字段2':字段值}的json对象或者字符串（where为json对象时只支持and的拼接），groupBy可以为数组对象如['字段1','字段2']，orderBy可以为json对象如{'字段1':'desc','字段2':''}，limit可以为json对象如{start:10,total:20}也可以为[10,20]，它们都支持字符串，limit还支持数字（此时表示的即是total的值），freestyleFields表示那些字段对应的值不需要做合法性校验和escape的自由字段(可支持如['字段1']的数组或者以逗号间隔的字符串)，其作用在where中有效。
  * ②可选参数cb：执行sql之后的回调函数，该回调函数的参数同mysql包的query方法中cb的参数。
* 7)table对象update数据，function table.update(options,cb) update方法支持2个参数：
  * ①必填参数options：{data:...,fields:...,values:...,where:...,freestyleFields:...}，data优先与field和values，data为如{'字段':字段值}的json对象，fields和values配套使用，其中fields表示字段(可支持如['字段1']的数组或者以逗号间隔的字符串)，values表示对应的值(可支持如['字段值','字段值']的数组)，where支持如{'字段1':字段值,'字段2':字段值}的json对象或者字符串（where为json对象时只支持and的拼接），freestyleFields表示那些字段对应的值不需要做合法性校验和escape的自由字段(可支持如['字段1']的数组或者以逗号间隔的字符串)，其作用在data、values和where中有效。
  * ②可选参数cb：执行sql之后的回调函数，该回调函数的参数同mysql包的query方法中cb的参数。
* 8)table对象delete数据，function table.delete(options,cb) delete方法支持2个参数：
  * ①可选参数options：{where:...,freestyleFields:...}，where支持如{'字段1':字段值,'字段2':字段值}的json对象或者字符串（where为json对象时只支持and的拼接），freestyleFields表示那些字段对应的值不需要做合法性校验和escape的自由字段(可支持如['字段1']的数组或者以逗号间隔的字符串)，其作用在where中有效。
  * ②可选参数cb：执行sql之后的回调函数，该回调函数的参数同mysql包的query方法中cb的参数。

###1.table对象获得途径

从默认数据库中获得一个table对象：
```js
var testTable = iMysql.table('test_table');
```

指定从某个数据库中获得一个table对象：
```js
var testTable = iMysql.table(1,'test_table');
```

动态指定从某个数据库中获得一个table对象：
```js
function chooseDb(){
    return 1;
}

var testTable = iMysql.table(chooseDb,'test_table');
```

动态指定表名获得一个table对象：
```js
function chooseTableName(){
    return 'test_table';
}

var testTable = iMysql.table(1,chooseTableName);
```

###2.table对象获取表名

```js
var testTable = iMysql.table('test_table');
console.log(testTable.getTableName());
```

###3.table对象获取当下所在的数据库索引

```js
var testTable = iMysql.table('test_table');
console.log(testTable.getDbIndex());
```

###4.table对象切换数据库

```js
var testTable = iMysql.table('test_table');
testTable.switch(0);
```

###5.table对象执行insert

```js
var testTable = iMysql.table('test_table');
//使用data插入(注意其中的freestyleFields的用法，它指定了data中的create_time字段的值是自由的即i-mysql是不会对它进行值合法性校验和escape的，table的insert、select、update、delete中都支持类似的freestyleFields用法，详细请看之前table对象中的介绍)
testTable.insert({data:{id:1,c1:'t1',create_time:'CURDATE()'},freestyleFields:'create_time'},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
//配套使用fields和values插入
testTable.insert({fields:['id','c1','create_time'],values:[2,'t2','CURDATE()'],freestyleFields:['create_time']},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
//批量插入
testTable.insert({fields:['id','c1','create_time'],values:[[3,'t3','CURDATE()'],[4,'t4','CURDATE()']],freestyleFields:'create_time'},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###6.table对象执行select

```js
var testTable = iMysql.table('test_table');
//查询所有数据
testTable.select(function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
//一个复杂的查询
testTable.select({fieldset:'count(*) as countNum',fields:['id','c1'],where:'id<100',groupBy:{c1:'desc'},limit:{start:2,total:100}},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###7.table对象执行update

```js
var testTable = iMysql.table('test_table');
//使用data修改
testTable.update({data:{c1:'update_value1'},where:{id:3}},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
//配套使用fields和values修改
testTable.update({fields:['id','c1'],values:[2,'update_value2'],where:{id:3}},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###8.table对象执行delete

```js
var testTable = iMysql.table('test_table');
testTable.delete({where:{id:3}},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###9.table对象连缀写法

```js
var testTable = iMysql.table('test_table');
testTable.insert({data:{id:1,c1:'t1'}},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
}).switch(1).select(function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```


##五、transaction对象
* 1)获取transaction对象，function transaction(dbIndex,transactionId) transaction方法可接受2个参数，dbIndex和transactionId（transactionId参数比较特殊，如果为负数，那么i-mysql会认为事务id不是去获取一个可能已经存在的事务，而是去获得一个全新的事务）都是可选的，如果只有1个参数传入则被认为该参数表示的是transactionId，其中dbIndex支持数字型参数或者一个返回数字型数据的function参数(dbIndex默认为i-mysql的默认数据库索引)，transactionId可以是事务id或者一个可以返回事务id数据的方法。
* 2)从transaction对象获取事务id，function transaction.getId()。
* 3)从transaction对象获取所在数据库索引，function transaction.getDbIndex()。
* 4)transaction对象获取是否超时或者设置事务超时时的自动提交时间并返回是否超时，function transaction.autoCommit(autoCommitTimeout) autoCommit方法如传入参数时表示需要设置超时自动提交时间（单位为毫秒）并返回事务在当下是否已经超时(boolean)，未传入参数（默认为10秒）则只返回事务在当下是否已经超时(boolean)。
* 5)transaction对象获取是否应该被销毁或者设置应该被销毁的闲置持续时间并返回是否应该被销毁，function transaction.destroy(destroyTimeout) destroy方法如传入参数时表示需要设置应该被销毁的闲置持续时间（单位为毫秒）并返回在当下是否已经应该被销毁(boolean)，未传入参数（默认为10分钟）则只返回在当下是否已经应该被销毁(boolean)。
* 6)transaction对象切换数据库，function transaction.switch(dbIndex,forceNewTransaction) switch方法接受2个参数，第一个参数dbIndex支持数字型参数或者一个返回数字型数据的function参数，switch在首次switch之后，只有在调用过commit或者rollback之后才能再次switch成不同的数据库（switch成同一个数据库多次则忽略），第二个参数forceNewTransaction(boolean)为真时，switch之后的事务将是一个全新的事务。
* 7)transaction对象事务提交，function transaction.commit(cb) commit方法支持传入一个回调函数，该函数与mysql包的commit方法的回调函数相同，当处于transaction的sql、insert、select、update、delete方法的回调函数中在不发生错误时调用则会强制提交。
* 8)transaction对象事务回滚，function transaction.rollback(cb) rollback方法支持传入一个回调函数，该函数与mysql包的rollback方法的回调函数相同，当处于transaction的sql、insert、select、update、delete方法的回调函数中在不发生错误时调用则会强制回滚。
* 9)transaction对象包装table，function transaction.table(tableObj_or_tableName) table方法支持table对象或者表示表名的字符串或者一个可以返回表名字符串数据或者table对象的方法，如传入为table对象时会校验它所在的数据库索引是否与当前事务所在的数据库索引一致，不一致则直接抛异常。
* 10)transaction对象包装table之后，则可以调用table的insert、select、update、delete方法，详细请参考之前的介绍，而在调用了transaction的sql方法之后就不能再次使用insert、select、update、delete这些方法了，只能通过再次调用transaction的table方法后才能再次使用。
* 11)transaction对象执行sql，function transaction.sql(sql,options,cb) sql方法可接受3个参数（与mysql包的query方法相同）：
  * ①必填参数sql：可以是字符串或者一个返回字符串数据的function。
  * ②可选参数options：用来填充sql的对象。
  * ③可选参数cb：执行sql之后的回调函数。

###1.transaction对象获得途径
注意：如果因业务需要您尝试通过事务id获取一个曾经获得过的transaction对象，那么请注意需要try catch，因为该事务如果不存在或者应该被销毁时会抛异常（destroy方法也许能帮助你解决部分问题）！实际上每次都获取一个全新的事务程序运行的效率反而高，所以不到万不得已请不要重复使用曾经的事务！

从默认数据库中获得一个全新的transaction对象：
```js
var trans1 = iMysql.transaction();
//或者
var trans1 = iMysql.transaction(-1);
```

在设置默认数据库后获得一个全新的transaction对象（不推荐这种写法，您可以尝试下面介绍的“从指定数据库中获得一个全新的transaction对象”的写法）：
```js
//如果只是临时改变默认数据库，那么请记得在调用transaction之后重新设置回原来的默认数据库
var oldDefaultDbIndex = iMysql.defaultDb();
var trans1 = iMysql.defaultDb(1).transaction();
iMysql.defaultDb(oldDefaultDbIndex);
```

从默认数据库中获得一个指定事务id的transaction对象（该事务可能是之前使用过的，但是如果该事务id的事务不存在或者已经被销毁则返回的是null）：
```js
var trans1 = iMysql.transaction(11);
```

从默认数据库中获得一个动态指定事务id的transaction对象：
```js
function chooseTransId(){
    return 23;
}

var trans1 = iMysql.transaction(chooseTransId);
```

从指定数据库中获得一个全新的transaction对象：
```js
var trans1 = iMysql.transaction(1,-1);
```

从指定数据库中获得一个指定事务id的transaction对象：
```js
var trans1 = iMysql.transaction(1,23);
```

从指定数据库中获得一个动态指定事务id的transaction对象：
```js
function chooseTransId(){
    return 23;
}

var trans1 = iMysql.transaction(1,chooseTransId);
```

###2.transaction对象获取事务id

```js
var trans1 = iMysql.transaction();
console.log(trans1.getId());
```

###3.transaction对象获取当下所在的数据库索引

```js
var trans1 = iMysql.transaction();
console.log(trans1.getDbIndex());
```

###4.transaction对象自动提交

判断transaction对象事务是否超时：
```js
var trans1 = iMysql.transaction();
console.log(trans1.autoCommit());
```

设置transaction对象事务超时时的自动提交时间并返回是否超时：
```js
var trans1 = iMysql.transaction();
console.log(trans1.autoCommit(5000));
```

###5.transaction对象自动销毁

判断transaction对象是否应该被销毁：
```js
var trans1 = iMysql.transaction();
console.log(trans1.destroy());
```

设置transaction对象应该被销毁的闲置持续时间并返回是否应该被销毁：
```js
var trans1 = iMysql.transaction();
console.log(trans1.destroy(60000));
```

###6.transaction对象切换数据库

```js
var trans1 = iMysql.transaction();
trans1.switch(1);//如果该trans1对象是首次切换数据库到‘1’这个数据库索引，那么切换之后事务id会与切换之前的事务id不同，否则即为上次切换过的事务id

trans1.switch(1,true);//第二个参数为真时，表示无论如何都会获得一个全新的transaction对象，所以此时该对象的事务id必定变化了
```

###7.transaction对象提交事务

注意：一定要注意在一个事务代码书写到最后时加上commit或者rollback的调用（在使用async包这类同步方法时除外，因为您可能在中途commit或者rollback，i-mysql称之为强制commit和强制rollback，执行了这些强制方法之后，如果还想做事务操作建议重新获得一个全新的事务对象来调用相关事务方法或者通过process.nextTick等方法在下一瞬调用原事务对象的sql、或者table的相关方法等，否则不会执行强制方法之后调用的方法和回调），以保证事务的完整性，否则会在占用一段时间的连接后自动提交事务！
```js
var trans1 = iMysql.transaction();
trans1.sql('insert into test_table(c1)values(\'insert_value1\')',function(err){
    if(err){
        //SQL执行发生错误是不需要执行rollback的，因为i-mysql已经帮您自动回滚了
        console.log(err);
    }else{
        //如果因流程控制需要，此处也可以调用commit或者rollback，调用之后除非发生错误否则该事务中的后续步骤和回调将都不被执行（为什么存在后续步骤及其回调呢？因为是异步的，下面代码中的“trans1.commit”其实已经在堆栈中了）
        console.log(arguments);
    }
});
trans1.sql('insert into test_table(c1)values(\'insert_value2\')',function(err){
    if(err){
        //SQL执行发生错误是不需要执行rollback的，因为i-mysql已经帮您自动回滚了
        console.log(err);
    }else{
        //如果因流程控制需要，此处也可以调用commit或者rollback，调用之后除非发生错误否则该事务中的后续步骤和回调将都不被执行（为什么存在后续步骤及其回调呢？因为是异步的，下面代码中的“trans1.commit”其实已经在堆栈中了）
        console.log(arguments);
    }
});
trans1.commit(function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###8.transaction对象回滚事务

注意：i-mysql具备SQL执行错误时自动回滚的，当然在SQL执行未发生错误时您也可以手动强制回滚，强制回滚调用之后除非发生错误否则该事务中的后续步骤和回调将都不被执行！
```js
var trans1 = iMysql.transaction();
trans1.sql('insert into test_table(c1)values(\'insert_value1\')',function(err){
    if(err){
        //SQL执行发生错误是不需要执行rollback的，因为i-mysql已经帮您自动回滚了
        console.log(err);
    }else{
        //如果因流程控制需要，此处也可以调用commit或者rollback，调用之后除非发生错误否则该事务中的后续步骤和回调将都不被执行（为什么存在后续步骤及其回调呢？因为是异步的，下面代码中的“trans1.commit”其实已经在堆栈中了）
        console.log(arguments);
    }
});
trans1.sql('insert into test_table(c1)values(\'insert_value2\')',function(err){
    if(err){
        //SQL执行发生错误是不需要执行rollback的，因为i-mysql已经帮您自动回滚了
        console.log(err);
    }else{
        console.log(arguments);
        //在此处强制回滚，调用之后除非发生错误否则该事务中的后续步骤和回调将都不被执行（为什么存在后续步骤及其回调呢？因为是异步的，下面代码中的“trans1.commit”其实已经在堆栈中了）
        this.rollback(function(err){
            if(err){
                console.log(err);
            }else{
                console.log(arguments);
            }
        });
    }
});
trans1.sql('insert into test_table(c1)values(\'insert_value3\')',function(err){
    if(err){
        //SQL执行发生错误是不需要执行rollback的，因为i-mysql已经帮您自动回滚了
        console.log(err);
    }else{
        //如果因流程控制需要，此处也可以调用commit或者rollback，调用之后除非发生错误否则该事务中的后续步骤和回调将都不被执行（为什么存在后续步骤及其回调呢？因为是异步的，下面代码中的“trans1.commit”其实已经在堆栈中了）
        console.log(arguments);
    }
});
trans1.commit(function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###9.transaction对象包装table

transaction对象使用table对象进行包装：
```js
var testTable = iMysql.table('test_table');
var trans1 = iMysql.transaction();
trans1.table(testTable);
```

transaction对象使用tableName进行包装：
```js
var trans1 = iMysql.transaction();
trans1.table('test_table');
```

transaction对象动态表名进行包装：
```js
function chooseTableName(){
    return 'test_table';
}
var trans1 = iMysql.transaction();
trans1.table(chooseTableName);
```

transaction对象动态表对象进行包装：
```js
function chooseTable(){
    return iMysql.table('test_table');
}
var trans1 = iMysql.transaction();
trans1.table(chooseTable);
```

###10.transaction对象使用table对象的insert、select、update、delete

注意：在调用了transaction的sql方法之后就不能再次使用insert、select、update、delete这些方法了，只能通过再次调用transaction的table方法后才能再次使用！
```js
var trans1 = iMysql.transaction();
trans1.table('test_table');
trans1.insert({data:{c1:'trans_insert_value1'}},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
trans1.commit(function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###11.transaction对象执行sql

```js
var trans1 = iMysql.transaction();
trans1.sql('insert into test_table(c1)values(\'insert_value4\')',function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
trans1.commit(function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```

###12.transaction对象连缀写法

```js
var trans1 = iMysql.transaction();
trans1.switch(1).sql('insert into test_table(c1)values(\'insert_value1\')',function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
}).sql('insert into test_table(c1)values(\'insert_value2\')',function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
}).table('test_table').insert({data:{c1:'trans_insert_value1'}},function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
}).sql('insert into test_table(c1)values(\'insert_value3\')',function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
}).commit(function(err){
    if(err){
        console.log(err);
    }else{
        console.log(arguments);
    }
});
```