---
layout: post
title: "3 unit tests to avoid bad surprises"
comments: true
visible: 1
---

On the road of continuous delivery, an essential stop is unit testing. They should be short, quick and reliable. Sometimes they are our only way to see an error and avoid to deliver a bug in production. This article presents 3 unit tests whose goal is to avoid bad surprises by focusing on key aspects of an Android application: Permissions, shared preferences and SQLite database. Check them out and avoid bad surprises on release day !

<!-- more -->

## Control your permissions

Managing permissions is often a key in the success of an application. We all know examples about application having a bad buzz because of abuses. On Android, users are very careful about it when installing a new application. Indeed, if they feel like you don't really need the permissions you are requesting, your conversion rate (visit on PlayStore / applications installed) can drop terribly.

Sometimes, when you add a new library if you don't pay attention, it can add permissions you don't need/want (I am looking at you Play Service…) and you will see your mistakes only when uploading your APK on the Play Store. Here is a unit test I wrote to avoid such unpleasant situation:

{% highlight java %}
@RunWith(RobolectricTestRunner.class)
@Config(manifest = Config.NONE)
public final class PermissionsTest {

    private static final String[] EXPECTED_PERMISSIONS = {
            […]
    };

    private static final String MERGED_MANIFEST =
        "build/intermediates/manifests/full/debug/AndroidManifest.xml"

    @Test
    public void shouldMatchPermissions() {
        AndroidManifest manifest = new AndroidManifest(
                Fs.fileFromPath(MERGED_MANIFEST),
                null,
                null
        );

        assertThat(new HashSet<>(manifest.getUsedPermissions())).
                containsOnly(EXPECTED_PERMISSIONS);
    }
}
{% endhighlight %}

This test is based on Robolectric for parsing an Android manifest. When Gradle builds an APK, one of the its step is to assemble all the manifests from the libraries you are using and merge them together. Then, this manifest is packaged into the binary. This test will look at the merged Android manifest, extract the permissions and verify that they match the expected permissions.

One drawback is that when you want to add a new permission for real, you also have to update the unit test. I agree that this is not ideal however sometimes you have to trade off to be safe. This is especially mandatory when your goal is continuous delivery (see my last [article](http://jeremie-martinez.com/2016/01/14/devops-on-android/)) and you want to be sure your permissions will not change.

## Validate your SharedPreferences

Most applications use `SharedPreferences` to store data. They are a core part and therefore must be heavily tested. To illustrate this example, I designed a small `SharedPreferences` wrapper, I am pretty sure you all have something similar in your app.

{% highlight java %}
public class Preferences {

  private final Context context;

  public Preferences(Context context) {
    this.context = context;
  }

  public String getUsername() {
    return getPreferences().getString("USERNAME", null);
  }

  public void setUsername(String username) {
    getPreferences().edit().
                     putString("USERNAME", username).
                     commit();
  }

  public boolean hasNotificationEnabled() {
    return getPreferences().getBoolean("NOTIFICATION"), false);
  }

  public void setNotificationEnabled(boolean enable) {
    return getPreferences().edit().
                            putBoolean("NOTIFICATION", enable).
                            commit();
  }

  private SharedPreferences getPreferences() {
    return context.getSharedPreferences(USER_PREFERENCES, MODE_PRIVATE);
  }
}
{% endhighlight %}

Thanks to Robolectric, it is pretty easy to test them:

{% highlight java %}
@RunWith(RobolectricTestRunner.class)
@Config(manifest = Config.NONE)
public final class PreferencesTest {

  private Preferences preferences;

  @Before
  public void setUp() {
    preferences = new Preferences(RuntimeEnvironment.application);
  }

  @Test
  public void should_set_username() {
    preferences.setUsername("jmartinez");
    assertThat(preferences.getUsername()).isEqualTo("jmartinez");
  }

  @Test
  public void should_set_notification() {
    preferences.setNotificationEnabled(true);
    assertThat(preferences.hasNotificationEnabled()).isTrue();
  }

  @Test
  public void should_match_defaults() {
    assertThat(preferences.getUsername()).isNull();
    assertThat(preferences.hasNotificationEnabled()).isFalse();
  }
}
{% endhighlight %}

Obviously, this is a simple example. However sometimes you can have more complex needs like serializing an object in JSON and store it in the `SharedPreferences`, or your wrapper can encapsulate more logic features (one `SharedPreferences` by user, several objects to store, …). In any case, testing your `SharedPreferences` should not be underestimated and neglected.

## Master database upgrades

Maintaining your SQLite database can be difficult. Indeed, your database will evolve with your application and making sure these migrations go well is mandatory. If you fail to do so, it will lead to crashes and losing users… Unacceptable !

This unit test is based on the work of an old colleague of mine [Thibaut](https://twitter.com/fredszaq). The idea is to compare the schema of a created from scratch database and an upgraded one.

{% highlight java %}
@RunWith(RobolectricTestRunner.class)
@Config(manifest = Config.NONE)
public final class MigrationTest {

  private File newFile;
  private File upgradedFile;

  @Before
  public void setup() throws IOException {
    File baseDir = new File("build/tmp/migration");
    newFile = new File(baseDir, "new.db");
    upgradedFile = new File(baseDir, "upgraded.db");
    File firstDbFile = new File("src/test/resources/origin.db");
    FileUtils.copyFile(firstDbFile, upgradedFile);
  }

  @Test
  public void upgradeShouldBeTheSameAsCreate() throws Exception {
    Context context = RuntimeEnvironment.application;
    DatabaseOpenHelper helper = new DatabaseOpenHelper(context);

    SQLiteDatabase newDb = SQLiteDatabase.openOrCreateDatabase(newFile, null);
    SQLiteDatabase upgradedDb = SQLiteDatabase.openDatabase(
            upgradedFile.getAbsolutePath(),
            null,
            SQLiteDatabase.OPEN_READWRITE
    );

    helper.onCreate(newDb);
    helper.onUpgrade(upgradedDb, 1, DatabaseOpenHelper.DATABASE_VERSION);

    Set<String> newSchema = extractSchema(newDbFile.getAbsolutePath());
    Set<String> upgradedSchema = extractSchema(upgradedDbFile.getAbsolutePath());

    assertThat(upgradedSchema).isEqualTo(newSchema);
  }

  private Set<String> extractSchema(String url) throws Exception {
    Connection conn = null;

    final Set<String> schema = new TreeSet<>();
    ResultSet tables = null;
    ResultSet columns = null

    try {
      conn = DriverManager.getConnection("jdbc:sqlite:" + url);

      tables = conn.getMetaData().getTables(null, null, null, null);
      while (tables.next()) {

        String tableName = tables.getString("TABLE_NAME");
        String tableType = tables.getString("TABLE_TYPE");
        schema.add(tableType + " " + tableName);

        columns = conn.getMetaData().getColumns(null, null, tableName, null);
              while (columns.next()) {

                String columnName = columns.getString("COLUMN_NAME");
                String columnType = columns.getString("TYPE_NAME");
                String columnNullable = columns.getString("IS_NULLABLE");
                String columnDefault = columns.getString("COLUMN_DEF");
                schema.add("TABLE " + tableName +
                        " COLUMN " + columnName + " " + columnType +
                        " NULLABLE=" + columnNullable +
                        " DEFAULT=" + columnDefault);
            }
        }

        return schema;
      } finally {
          closeQuietly(tables);
          closeQuietly(columns);
          closeQuietly(conn);
      }
    }
}
{% endhighlight %}

 The approach is pretty straight forward. For each database:

 1. We iterate on the tables
 2. We build a string representing each table
 3. We iterate on every column of the table
 4. We build a string representing the column

These strings represent our database schema. Finally, we compare the two schemas that should be identical.

This is just an example but this schema can be extended since the API offers much more items available. You can see what is possible from the following documentation: [Metadata](https://docs.oracle.com/javase/7/docs/api/java/sql/DatabaseMetaData.html). For instance, you could also compare references or indexes. Once again, apply what suits best to your app.

Database migration is very important and unfortunately it is often a source of bugs. This unit test will help you making sure your migration works and you can therefore upgrade safely.

## Conclusion

These unit tests are just examples, however I hope this article showed you that a lot can be achieved with them. To reach continuous delivery, knowing you are safe about database migration, permissions or SharedPreferences is a huge advantage.
