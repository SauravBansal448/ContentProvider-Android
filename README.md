# ContentProvider-Android


## Content Provider Example

*This Part is for insert user data and load data from database*

#### Step.1 - MyContentProvider.class

1. Creating the Content Provider class(Click on File, then New => Other => ContentProvider)
2. Click on File, then New => Other => ContentProvider.
3.Name the ContentProvider
4. Define authority (it can be anything for example “com.demo.user.provider”)
5. Select Exported and Enabled option


```
package com.example.contentprovidersinandroid 

import android.content.* 
import android.database.Cursor 
import android.database.sqlite.SQLiteDatabase 
import android.database.sqlite.SQLiteException 
import android.database.sqlite.SQLiteOpenHelper 
import android.database.sqlite.SQLiteQueryBuilder 
import android.net.Uri 


class MyContentProvider : ContentProvider() { 
	companion object { 
		// defining authority so that other application can access it 
		const val PROVIDER_NAME = "com.demo.user.provider"

		// defining content URI 
		const val URL = "content://$PROVIDER_NAME/users"

		// parsing the content URI 
		val CONTENT_URI = Uri.parse(URL) 
		const val id = "id"
		const val name = "name"
		const val uriCode = 1
		var uriMatcher: UriMatcher? = null
		private val values: HashMap<String, String>? = null

		// declaring name of the database 
		const val DATABASE_NAME = "UserDB"

		// declaring table name of the database 
		const val TABLE_NAME = "Users"

		// declaring version of the database 
		const val DATABASE_VERSION = 1

		// sql query to create the table 
		const val CREATE_DB_TABLE = 
			(" CREATE TABLE " + TABLE_NAME 
					+ " (id INTEGER PRIMARY KEY AUTOINCREMENT, "
					+ " name TEXT NOT NULL);") 

		init { 

			// to match the content URI 
			// every time user access table under content provider 
			uriMatcher = UriMatcher(UriMatcher.NO_MATCH) 

			// to access whole table 
			uriMatcher!!.addURI( 
				PROVIDER_NAME, 
				"users", 
				uriCode 
			) 

			// to access a particular row 
			// of the table 
			uriMatcher!!.addURI( 
				PROVIDER_NAME, 
				"users/*", 
				uriCode 
			) 
		} 
	} 

	override fun getType(uri: Uri): String? { 
		return when (uriMatcher!!.match(uri)) { 
			uriCode -> "vnd.android.cursor.dir/users"
			else -> throw IllegalArgumentException("Unsupported URI: $uri") 
		} 
	} 

	// creating the database 
	override fun onCreate(): Boolean { 
		val context = context 
		val dbHelper = 
			DatabaseHelper(context) 
		db = dbHelper.writableDatabase 
		return if (db != null) { 
			true
		} else false
	} 

	override fun query( 
		uri: Uri, projection: Array<String>?, selection: String?, 
		selectionArgs: Array<String>?, sortOrder: String? 
	): Cursor? { 
		var sortOrder = sortOrder 
		val qb = SQLiteQueryBuilder() 
		qb.tables = TABLE_NAME 
		when (uriMatcher!!.match(uri)) { 
			uriCode -> qb.projectionMap = values 
			else -> throw IllegalArgumentException("Unknown URI $uri") 
		} 
		if (sortOrder == null || sortOrder === "") { 
			sortOrder = id 
		} 
		val c = qb.query( 
			db, projection, selection, selectionArgs, null, 
			null, sortOrder 
		) 
		c.setNotificationUri(context!!.contentResolver, uri) 
		return c 
	} 

	// adding data to the database 
	override fun insert(uri: Uri, values: ContentValues?): Uri? { 
		val rowID = db!!.insert(TABLE_NAME, "", values) 
		if (rowID > 0) { 
			val _uri = 
				ContentUris.withAppendedId(CONTENT_URI, rowID) 
			context!!.contentResolver.notifyChange(_uri, null) 
			return _uri 
		} 
		throw SQLiteException("Failed to add a record into $uri") 
	} 

	override fun update( 
		uri: Uri, values: ContentValues?, selection: String?, 
		selectionArgs: Array<String>? 
	): Int { 
		var count = 0
		count = when (uriMatcher!!.match(uri)) { 
			uriCode -> db!!.update(TABLE_NAME, values, selection, selectionArgs) 
			else -> throw IllegalArgumentException("Unknown URI $uri") 
		} 
		context!!.contentResolver.notifyChange(uri, null) 
		return count 
	} 

	override fun delete( 
		uri: Uri, 
		selection: String?, 
		selectionArgs: Array<String>? 
	): Int { 
		var count = 0
		count = when (uriMatcher!!.match(uri)) { 
			uriCode -> db!!.delete(TABLE_NAME, selection, selectionArgs) 
			else -> throw IllegalArgumentException("Unknown URI $uri") 
		} 
		context!!.contentResolver.notifyChange(uri, null) 
		return count 
	} 

	// creating object of database 
	// to perform query 
	private var db: SQLiteDatabase? = null

	// creating a database 
	private class DatabaseHelper // defining a constructor 
	internal constructor(context: Context?) : SQLiteOpenHelper( 
		context, 
		DATABASE_NAME, 
		null, 
		DATABASE_VERSION 
	) { 
		// creating a table in the database 
		override fun onCreate(db: SQLiteDatabase) { 
			db.execSQL(CREATE_DB_TABLE) 
		} 

		override fun onUpgrade( 
			db: SQLiteDatabase, 
			oldVersion: Int, 
			newVersion: Int 
		) { 

			// sql query to drop a table 
			// having similar name 
			db.execSQL("DROP TABLE IF EXISTS $TABLE_NAME") 
			onCreate(db) 
		} 
	} 
}
```

#### Step-2  Design the activity_main.xml layout
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#DDEEFF"
    tools:context=".MainActivity">

    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/tvAppName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="30dp"
        android:layout_marginTop="25dp"
        android:gravity="center"
        android:text="@string/app_name"
        android:textColor="#000000"
        android:textSize="40sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <androidx.appcompat.widget.AppCompatEditText
        android:id="@+id/textName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="20dp"
        android:layout_marginTop="20dp"
        android:background="#d4d4d4"
        android:hint="@string/hintText"
        android:paddingVertical="10dp"
        android:paddingStart="10dp"
        android:textColorHint="@color/black"
        app:layout_constraintTop_toBottomOf="@id/tvAppName" />


    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/insertButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginTop="25dp"
        android:layout_marginEnd="20dp"
        android:layout_marginBottom="20dp"
        android:background="#4CAF50"
        android:text="@string/insert"
        android:textAlignment="center"
        android:textAppearance="@style/TextAppearance.AppCompat.Display1"
        android:textColor="#FFFFFF"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/textName" />

    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/loadButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginTop="10dp"
        android:layout_marginEnd="20dp"
        android:layout_marginBottom="20dp"
        android:background="#4CAF50"
        android:text="@string/loadButtonText"
        android:textAlignment="center"
        android:textAppearance="@style/TextAppearance.AppCompat.Display1"
        android:textColor="#FFFFFF"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/insertButton" />


    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/res"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginEnd="20dp"
        android:clickable="false"
        android:ems="10"
        android:textColor="@android:color/holo_green_dark"
        android:textSize="18sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/loadButton" />

</androidx.constraintlayout.widget.ConstraintLayout>

```

#### Step-3  Modify the MainActivity.class
```
*Add Details*
private fun addUserDetails() {
        // class to add values in the database
        val values = ContentValues()
        // fetching text from user
        values.put(MyContentProvider.name, (findViewById<View>(R.id.textName) as EditText).text.toString())
        // inserting into database through content URI
        contentResolver.insert(MyContentProvider.CONTENT_URI, values)
        // displaying a toast message
        Toast.makeText(baseContext, "New Record Inserted", Toast.LENGTH_LONG).show()
    }
    
```


```
*Load Details*
private fun loadUserDetails() {
        // inserting complete table details in this text field
        val resultView = findViewById<View>(R.id.res) as TextView

        // creating a cursor object of the
        // content URI
        val cursor = contentResolver.query(Uri.parse("content://com.demo.user.provider/users"), null, null, null, null)

        // iteration of the cursor
        // to print whole table
        if (cursor!!.moveToFirst()) {
            val strBuild = StringBuilder()
            while (!cursor.isAfterLast) {
                strBuild.append(""" 
      
    ${cursor.getString(cursor.getColumnIndex("id"))}-${cursor.getString(cursor.getColumnIndex("name"))} 
    """.trimIndent())
                cursor.moveToNext()
            }
            resultView.text = strBuild
        } else {
            resultView.text = "No Records Found"
        }
    }
```

#### Step 4- Modify the AndroidManifest file

```
 <provider
            android:name="com.example.contentprovidersinandroid.MyContentProvider"
            android:authorities="com.demo.user.provider"
            android:enabled="true"
            android:exported="true"/>
```


## Content Provider Receiver Example

*This part is for receiving data*

#### Step-1: Designing the activity_main.xml layout

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#DDEEFF"
    tools:context=".MainActivity">

    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/TextView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="40dp"
        android:gravity="center"
        android:text="@string/heading"
        android:textColor="@android:color/holo_green_dark"
        android:textSize="36sp"
        android:textStyle="bold"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />


    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/loadButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginTop="40dp"
        android:layout_marginEnd="20dp"
        android:layout_marginBottom="20dp"
        android:background="#4CAF50"
        android:text="@string/loadButtonText"
        android:textAlignment="center"
        android:textAppearance="@style/TextAppearance.AppCompat.Display1"
        android:textColor="#FFFFFF"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/TextView1" />


    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/res"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginTop="20dp"
        android:layout_marginEnd="20dp"
        android:clickable="false"
        android:ems="10"
        android:textColor="@android:color/holo_green_dark"
        android:textSize="18sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/loadButton" />


</androidx.constraintlayout.widget.ConstraintLayout>

```
#### Step-2 Modify the MainActivity file

```
*loadData() in OnCreate()


 private fun loadData() {
        // inserting complete table details in this text field

        // creating a cursor object of the
        // content URI
        val cursor = contentResolver.query(
            CONTENT_URI,
            null,
            null,
            null,
            null
        )

        // iteration of the cursor
        // to print whole table
        if (cursor != null) {
            if (cursor.moveToFirst()) {
                val strBuild = StringBuilder()
                while (!cursor.isAfterLast) {
                    strBuild.append(
                        "\n" + cursor.getString(cursor.getColumnIndex("id")) + "-" + cursor.getString(
                            cursor.getColumnIndex("name")
                        )
                    )
                    cursor.moveToNext()
                }
                res.text = strBuild
            } else {
                res.text = "No Records Found"
            }
        }
    }

```
