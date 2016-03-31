## Reverse

burn_in_hell_creator (400)

В .rar архиве нам выдают содержимое андроид-приложения (исходники, конфиги, ресурсы). Увидев в корне sqlite3 бд database.db, не глядя сначала в код, можно предположить, что там лежит если не флаг, то, по меньшей мере, что-то непосредственно с ним связанное. В дампе видим следующее:  
```sql
BEGIN TRANSACTION;
CREATE TABLE "android_metadata" ("locale" TEXT DEFAULT 'en_US');
INSERT INTO "android_metadata" VALUES('en_US');
CREATE TABLE img("bytes" TEXT);
INSERT INTO "img" VALUES('[B@53a4e6ac');
INSERT INTO "img" VALUES('[B@53a4e660');
INSERT INTO "img" VALUES('[B@53a54a0c');
CREATE TABLE "jokes" ("joke" TEXT);
INSERT INTO "jokes" VALUES('Мменя зовут Саша, но друзья зовут меня выпить');
INSERT INTO "jokes" VALUES('пьяной дракой закончилась, начавшаяся пьяной дракой драка');
...
INSERT INTO "jokes" VALUES('наркотики - нарпёсики');
COMMIT;
```
Из интересного здесь разве что таблица img. Погуглив, можно понять, что это всего лишь обозначение object ID в java, в данном случае это byte array. Содержимого изображений в базе, к сожалению, нет. Посмотрим в код. Часть DataBaseHelper.java выглядит так:
```java
public class DataBaseHelper extends SQLiteOpenHelper  {

    // путь к базе данных вашего приложения
    private static String DB_PATH = "/data/data/com.example.nester.androidtask/databases/";

    private static String DB_NAME = "database.db";
    private SQLiteDatabase myDataBase;
    private final Context mContext;

    private static final String TABLE_JOKES = "jokes";
    private static final String TABLE_SECRET_DATA = "img";

    public static final String COLUMN_JOKE = "joke";
    public static final String COLUMN_DATA = "bytes";
...
    public String getSecretData(){
        SQLiteDatabase db;
        try {
            db = this.getReadableDatabase();

            Cursor cursor = db.query(TABLE_SECRET_DATA, null, null, null, null, null, null);

            ArrayList<String> buffer = new ArrayList<>();
            cursor.getCount();


            for(int i=0;i<cursor.getCount() && cursor!=null ;i++) {
                if (cursor.moveToFirst()) {
                    cursor.move(i);
                    buffer.add(cursor.getString(cursor.getColumnIndex(COLUMN_DATA)));
                }
            }

            cursor.close();
            db.close();
            return buffer.get(buffer.size() - 1);
        }
        catch (SQLiteException ex)
        {
            throw new Error("Unable to get names of lists");
        }
    }
...
```  
MainActivity.java:
```java
...
    public static byte[] getBytes(Bitmap bitmap) {
        ByteArrayOutputStream stream = new ByteArrayOutputStream();
        bitmap.compress(Bitmap.CompressFormat.JPEG, 0, stream);
        return stream.toByteArray();
    }
...
```
TABLE_SECRET_DATA как бы намекает нам, что img это именно то, что нужно, но, как видно, в базе ничего нет. Предположение о том, где может быть флаг, по-моему, может появится только если связать getBytes и имя таблицы в TABLE_SECRET_DATA, т.е. содержанием img могли бы быть jpg файлы, поискав которые, найдем:  
```
$ find . -type f -name "*\.jpg"
./app/build/intermediates/res/merged/debug/drawable/tities.jpg
```    
![x](http://i.imgur.com/xADjUtT.png "flag")