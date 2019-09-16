### 基本步骤  

◆ 继承 ContentProvider    
public class DownLoadContentProvider extends ContentProvider  

◆ 配置清单文件  
```
<provider
    android:name="com.alex.andfun.service.huggles.provider.DownLoadContentProvider"+
    android:authorities="com.alex.andfun.service.provider.DownLoadContentProvider"
    android:exported="true"
    android:enabled="true"
    tools:ignore="ExportedContentProvider" />
```

◆ 配置uriMatcher  
```
public class DownLoadContentProvider extends ContentProvider {

    private SQLiteDatabase sqLiteDatabase;
    private static UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

    static {
        uriMatcher.addURI(C.ProviderAction.DOWNLOAD_AUTHORITY, C.Database.getUserTableEntity().getTableName(), C.Database.getUserTableEntity().getMatchCode());
        uriMatcher.addURI(C.ProviderAction.DOWNLOAD_AUTHORITY, C.Database.getOrderTableEntity().getTableName(), C.Database.getOrderTableEntity().getMatchCode());
        uriMatcher.addURI(C.ProviderAction.DOWNLOAD_AUTHORITY, C.Database.getMessageTableEntity().getTableName(), C.Database.getMessageTableEntity().getMatchCode());
    }
}
```

◆ 配置SQLiteDatabase  
```
/**
 * 只会被调用一次，数据初始化操作
 */
@Override
public boolean onCreate() {
    DBOpenHelper dbOpenHelper = new DBOpenHelper(getContext());
    sqLiteDatabase = dbOpenHelper.getWritableDatabase();
    return false;
}
```

#### Insert  

```
@SuppressWarnings("ConstantConditions")
@Nullable
@Override
public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
    int match = uriMatcher.match(uri);
    String tableName = C.Database.getMatchMap().get(match);
    LogTrack.w("tableName = " + tableName);
    sqLiteDatabase.insert(tableName, null, values);
    /*当该URI的ContentProvider数据发生变化时，通知外界（即访问该ContentProvider数据的访问者）*/
    ContentResolver contentResolver = getContext().getContentResolver();
    if (contentResolver != null) {
        contentResolver.notifyChange(uri, null);
    }
    return uri;
}
```

