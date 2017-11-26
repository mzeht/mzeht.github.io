title: greendao 数据库升级
categories:
  - 移动开发
  - Android
author: mzeht
avatar: /images/favicon.png
authorDesc: 一个写代码的
date: 2017-05-28 15:47:26
tags:
keywords:
description:
photos:
---

创建一个数据库帮助类


```
public class MyOpenHelper extends DaoMaster.OpenHelper{
    public MyOpenHelper(Context context, String name) {
        super(context, name);
    }

    public MyOpenHelper(Context context, String name, SQLiteDatabase.CursorFactory factory) {
        super(context, name, factory);
    }

    @Override
    public void onUpgrade(Database db, int oldVersion, int newVersion) {
        L.d("db version update from " + oldVersion + " to " + newVersion);
        switch (oldVersion){
            case 1:
                CollectAdditionDao.createTable(db,true);
                break;
        }
    }

}
```

修改greenDao的版本号，在内层的gradle中的buildTypes节点下添加


```
 greendao {
        // schemaVersion 1
        schemaVersion 2//@add 增值服务表
        targetGenDir 'src/main/java'
    }
```

更新方式

1.删除再新建


```
/**
     * 删除原表重新再建立一个表
     * @param db
     */
    public void dropAndCreate(Database db){
        DaoMaster.dropAllTables(db, true);
        DaoMaster.createAllTables(db, false);
    }
```

2.备份数据库，建立新数据库，然后将备份导入


```
/**
     * 备份还原
     * @param db
     * @param daoClasses
     */
    public void migrate(Database db, Class<? extends AbstractDao<?, ?>>... daoClasses) {
        generateTempTables(db, daoClasses);
        DaoMaster.dropAllTables(db, true);
        DaoMaster.createAllTables(db, false);
        restoreData(db, daoClasses);
    }
    /**
     * 数据库备份
     * @param db
     * @param daoClasses
     */
    private void generateTempTables(Database db, Class<? extends AbstractDao<?, ?>>... daoClasses) {
        for (int i = 0; i < daoClasses.length; i++) {
            DaoConfig daoConfig = new DaoConfig(db, daoClasses[i]);

            String divider = "";
            String tableName = daoConfig.tablename;
            String tempTableName = daoConfig.tablename.concat("_TEMP");
            ArrayList<String> properties = new ArrayList<>();

            StringBuilder createTableStringBuilder = new StringBuilder();

            createTableStringBuilder.append("CREATE TABLE ").append(tempTableName).append(" (");

            for (int j = 0; j < daoConfig.properties.length; j++) {
                String columnName = daoConfig.properties[j].columnName;

                if (getColumns(db, tableName).contains(columnName)) {
                    properties.add(columnName);

                    String type = null;

                    try {
                        type = getTypeByClass(daoConfig.properties[j].type);
                    } catch (Exception exception) {
                    }

                    createTableStringBuilder.append(divider).append(columnName).append(" ").append(type);

                    if (daoConfig.properties[j].primaryKey) {
                        createTableStringBuilder.append(" PRIMARY KEY");
                    }

                    divider = ",";
                }
            }
            createTableStringBuilder.append(");");

            db.execSQL(createTableStringBuilder.toString());

            StringBuilder insertTableStringBuilder = new StringBuilder();

            insertTableStringBuilder.append("INSERT INTO ").append(tempTableName).append(" (");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(") SELECT ");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(" FROM ").append(tableName).append(";");

            db.execSQL(insertTableStringBuilder.toString());
        }
    }
    /**
     * 数据库恢复
     * @param db
     * @param daoClasses
     */
    private void restoreData(Database db, Class<? extends AbstractDao<?, ?>>... daoClasses) {
        for (int i = 0; i < daoClasses.length; i++) {
            DaoConfig daoConfig = new DaoConfig(db, daoClasses[i]);

            String tableName = daoConfig.tablename;
            String tempTableName = daoConfig.tablename.concat("_TEMP");
            ArrayList<String> properties = new ArrayList();

            for (int j = 0; j < daoConfig.properties.length; j++) {
                String columnName = daoConfig.properties[j].columnName;

                if (getColumns(db, tempTableName).contains(columnName)) {
                    properties.add(columnName);
                }
            }

            StringBuilder insertTableStringBuilder = new StringBuilder();

            insertTableStringBuilder.append("INSERT INTO ").append(tableName).append(" (");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(") SELECT ");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(" FROM ").append(tempTableName).append(";");

            StringBuilder dropTableStringBuilder = new StringBuilder();

            dropTableStringBuilder.append("DROP TABLE ").append(tempTableName);

            db.execSQL(insertTableStringBuilder.toString());
            db.execSQL(dropTableStringBuilder.toString());
        }
    }
    private String getTypeByClass(Class<?> type) throws Exception {
        if (type.equals(String.class)) {
            return "TEXT";
        }
        if (type.equals(Long.class) || type.equals(Integer.class) || type.equals(long.class)) {
            return "INTEGER";
        }
        if (type.equals(Boolean.class)) {
            return "BOOLEAN";
        }

        Exception exception =
                new Exception(CONVERSION_CLASS_NOT_FOUND_EXCEPTION.concat(" - Class: ").concat(type.toString()));
        throw exception;
    }

    private static List<String> getColumns(Database db, String tableName) {
        List<String> columns = new ArrayList<>();
        Cursor cursor = null;
        try {
            cursor = db.rawQuery("SELECT * FROM " + tableName + " limit 1", null);
            if (cursor != null) {
                columns = new ArrayList<>(Arrays.asList(cursor.getColumnNames()));
            }
        } catch (Exception e) {
            Log.v(tableName, e.getMessage(), e);
            e.printStackTrace();
        } finally {
            if (cursor != null) cursor.close();
        }
        return columns;
    }
```

3.对比表差异，向原表中直接插入column

```
/**
     * 对比差异，在原表中直接添加column,赞不做删除操作
     * @param db
     * @param daoClasses
     */
    public void contrastDiff(Database db,ArrayList<String>properties, Class<? extends AbstractDao<?, ?>>... daoClasses){
     for(int i=0;i<daoClasses.length;i++){
         DaoConfig daoConfig = new DaoConfig(db, daoClasses[i]);
         String tableName=daoConfig.tablename;
         if(properties!=null&&properties.size()>0){
             ArrayList<String>tem=new ArrayList<>();
             StringBuilder sqlBuilder=new StringBuilder();
             for(int j=0;j<properties.size();j++){
                 if(getColumns(db,tableName).contains(properties.get(j))){
                     continue;
                 }
                 tem.add(properties.get(j));
             }
             sqlBuilder.append("INSERT INTO ").append(tableName).append(" (");
             sqlBuilder.append(TextUtils.join(",", tem));
             sqlBuilder.append(") SELECT ");
             sqlBuilder.append(TextUtils.join(",", tem));
             sqlBuilder.append(" FROM ").append(tableName).append(";");
             db.execSQL(sqlBuilder.toString());
         }
        }
    }
```


完整的数据库升级帮助类

```
/**
 * description: greenDao升级帮助
 * author: bear .
 * Created date:  2017/5/17.
 */
public class MigrationHelper {

    private static final String CONVERSION_CLASS_NOT_FOUND_EXCEPTION =
            "MIGRATION HELPER - CLASS DOESN'T MATCH WITH THE CURRENT PARAMETERS";
    private static MigrationHelper instance;

    public static MigrationHelper getInstance() {
        if (instance == null) {
            instance = new MigrationHelper();
        }
        return instance;
    }

    /**
     * 备份还原
     * @param db
     * @param daoClasses
     */
    public void migrate(Database db, Class<? extends AbstractDao<?, ?>>... daoClasses) {
        generateTempTables(db, daoClasses);
        DaoMaster.dropAllTables(db, true);
        DaoMaster.createAllTables(db, false);
        restoreData(db, daoClasses);
    }

    /**
     * 对比差异，在原表中直接添加column,赞不做删除操作
     * @param db
     * @param daoClasses
     */
    public void contrastDiff(Database db,ArrayList<String>properties, Class<? extends AbstractDao<?, ?>>... daoClasses){
     for(int i=0;i<daoClasses.length;i++){
         DaoConfig daoConfig = new DaoConfig(db, daoClasses[i]);
         String tableName=daoConfig.tablename;
         if(properties!=null&&properties.size()>0){
             ArrayList<String>tem=new ArrayList<>();
             StringBuilder sqlBuilder=new StringBuilder();
             for(int j=0;j<properties.size();j++){
                 if(getColumns(db,tableName).contains(properties.get(j))){
                     continue;
                 }
                 tem.add(properties.get(j));
             }
             sqlBuilder.append("INSERT INTO ").append(tableName).append(" (");
             sqlBuilder.append(TextUtils.join(",", tem));
             sqlBuilder.append(") SELECT ");
             sqlBuilder.append(TextUtils.join(",", tem));
             sqlBuilder.append(" FROM ").append(tableName).append(";");
             db.execSQL(sqlBuilder.toString());
         }
        }
    }

    /**
     * 删除原表重新再建立一个表
     * @param db
     */
    public void dropAndCreate(Database db){
        DaoMaster.dropAllTables(db, true);
        DaoMaster.createAllTables(db, false);
    }

    /**
     * 数据库备份
     * @param db
     * @param daoClasses
     */
    private void generateTempTables(Database db, Class<? extends AbstractDao<?, ?>>... daoClasses) {
        for (int i = 0; i < daoClasses.length; i++) {
            DaoConfig daoConfig = new DaoConfig(db, daoClasses[i]);

            String divider = "";
            String tableName = daoConfig.tablename;
            String tempTableName = daoConfig.tablename.concat("_TEMP");
            ArrayList<String> properties = new ArrayList<>();

            StringBuilder createTableStringBuilder = new StringBuilder();

            createTableStringBuilder.append("CREATE TABLE ").append(tempTableName).append(" (");

            for (int j = 0; j < daoConfig.properties.length; j++) {
                String columnName = daoConfig.properties[j].columnName;

                if (getColumns(db, tableName).contains(columnName)) {
                    properties.add(columnName);

                    String type = null;

                    try {
                        type = getTypeByClass(daoConfig.properties[j].type);
                    } catch (Exception exception) {
                    }

                    createTableStringBuilder.append(divider).append(columnName).append(" ").append(type);

                    if (daoConfig.properties[j].primaryKey) {
                        createTableStringBuilder.append(" PRIMARY KEY");
                    }

                    divider = ",";
                }
            }
            createTableStringBuilder.append(");");

            db.execSQL(createTableStringBuilder.toString());

            StringBuilder insertTableStringBuilder = new StringBuilder();

            insertTableStringBuilder.append("INSERT INTO ").append(tempTableName).append(" (");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(") SELECT ");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(" FROM ").append(tableName).append(";");

            db.execSQL(insertTableStringBuilder.toString());
        }
    }

    /**
     * 数据库恢复
     * @param db
     * @param daoClasses
     */
    private void restoreData(Database db, Class<? extends AbstractDao<?, ?>>... daoClasses) {
        for (int i = 0; i < daoClasses.length; i++) {
            DaoConfig daoConfig = new DaoConfig(db, daoClasses[i]);

            String tableName = daoConfig.tablename;
            String tempTableName = daoConfig.tablename.concat("_TEMP");
            ArrayList<String> properties = new ArrayList();

            for (int j = 0; j < daoConfig.properties.length; j++) {
                String columnName = daoConfig.properties[j].columnName;

                if (getColumns(db, tempTableName).contains(columnName)) {
                    properties.add(columnName);
                }
            }

            StringBuilder insertTableStringBuilder = new StringBuilder();

            insertTableStringBuilder.append("INSERT INTO ").append(tableName).append(" (");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(") SELECT ");
            insertTableStringBuilder.append(TextUtils.join(",", properties));
            insertTableStringBuilder.append(" FROM ").append(tempTableName).append(";");

            StringBuilder dropTableStringBuilder = new StringBuilder();

            dropTableStringBuilder.append("DROP TABLE ").append(tempTableName);

            db.execSQL(insertTableStringBuilder.toString());
            db.execSQL(dropTableStringBuilder.toString());
        }
    }

    private String getTypeByClass(Class<?> type) throws Exception {
        if (type.equals(String.class)) {
            return "TEXT";
        }
        if (type.equals(Long.class) || type.equals(Integer.class) || type.equals(long.class)) {
            return "INTEGER";
        }
        if (type.equals(Boolean.class)) {
            return "BOOLEAN";
        }

        Exception exception =
                new Exception(CONVERSION_CLASS_NOT_FOUND_EXCEPTION.concat(" - Class: ").concat(type.toString()));
        throw exception;
    }

    private static List<String> getColumns(Database db, String tableName) {
        List<String> columns = new ArrayList<>();
        Cursor cursor = null;
        try {
            cursor = db.rawQuery("SELECT * FROM " + tableName + " limit 1", null);
            if (cursor != null) {
                columns = new ArrayList<>(Arrays.asList(cursor.getColumnNames()));
            }
        } catch (Exception e) {
            Log.v(tableName, e.getMessage(), e);
            e.printStackTrace();
        } finally {
            if (cursor != null) cursor.close();
        }
        return columns;
    }
}
```






