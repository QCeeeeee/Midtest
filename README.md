# NotePad-Android应用的介绍文档

## 一.初始应用的功能

### 1.新建笔记和编辑笔记

(1)在主界面点击红色矩形所示按钮，新建笔记并进入编辑界面

<img width="411" height="582" alt="QQ_1765354950013" src="https://github.com/user-attachments/assets/1f66216c-c7e3-43d6-ae59-3d301d95de55" />

(2)进入笔记编辑界面后，可进行笔记编辑

<img width="421" height="783" alt="QQ_1765354969152" src="https://github.com/user-attachments/assets/868e1678-f0e7-428e-9047-965eb6249d62" />

### 2.保存和删除

(1)右上角第一个是保存按钮，点击后可保存笔记且显示保存成功3s，同时跳转主界面

<img width="434" height="321" alt="QQ_1765355137664" src="https://github.com/user-attachments/assets/50f9b122-762e-4f8c-b82d-a0bcb9e1d810" />

<img width="430" height="726" alt="QQ_1765355144023" src="https://github.com/user-attachments/assets/048625c3-9914-466c-930d-f828c36a9711" />


(2)右上角第二个是删除按钮，点击后跳出页面确认是否删除，点击删除后自动返回主界面

<img width="416" height="359" alt="QQ_1765355223361" src="https://github.com/user-attachments/assets/84c4f95f-fbdd-4ab8-9dd5-ba6b59307304" />

<img width="420" height="763" alt="QQ_1765355230410" src="https://github.com/user-attachments/assets/0b451975-cfdb-4924-a75b-803cc4bee486" />


### 3.笔记列表

在进行笔记的新建和编辑后，在主界面中呈现笔记列表。
<img width="462" height="573" alt="QQ_1765355331786" src="https://github.com/user-attachments/assets/704531d4-7137-48d5-a65d-40c149449078" />


## 二.拓展基本功能

### （一）.笔记条目增加时间戳显示

#### 1. 功能要求
- 每个新建笔记都会保存新建时间并显示
- 在修改笔记后更新为修改时间
- 在笔记列表中直观显示时间信息

#### 2. 实现思路和技术实现

**实现思路：**
1. 数据库层面：在笔记表中添加 `created` 和 `modified` 时间字段
2. 数据操作层面：插入时记录创建时间，更新时记录修改时间
3. 界面显示层面：在列表项中添加时间戳显示区域
4. 数据格式化：将时间戳转换为易读的日期时间格式

**技术实现：**

##### 2.1 数据库修改
在 `NotePadProvider.java` 的 `DatabaseHelper` 类中，创建表时包含时间字段：
```sql
CREATE TABLE notes (
    _id INTEGER PRIMARY KEY,
    title TEXT,
    note TEXT,
    created INTEGER,        -- 创建时间戳
    modified INTEGER        -- 修改时间戳
);
```
#### 2.2 插入和更新时间戳
在 insert() 和 update() 方法中自动设置时间：
```java
// 插入新笔记时
Long now = System.currentTimeMillis();
values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, now);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, now);

// 更新笔记时
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
```

#### 2.3 列表界面显示时间戳
修改 noteslist_item.xml 布局文件，添加时间戳显示：
```xml
<LinearLayout>
    <TextView android:id="@android:id/text1" ... />  <!-- 标题 -->
    <TextView android:id="@+id/text2" ... />          <!-- 时间戳 -->
</LinearLayout>
 ```     
#### 2.4 时间戳格式化显示
在 NotesList.java 中格式化时间戳：
```java
SimpleCursorAdapter adapter = new SimpleCursorAdapter(...) {
    @Override
    public void setViewText(TextView v, String text) {
        if (v.getId() == R.id.text2 && text != null) {
            try {
                long timeMillis = Long.parseLong(text);
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm", Locale.getDefault());
                text = sdf.format(new Date(timeMillis));
            } catch (NumberFormatException e) {
                // 保持原文本
            }
        }
        super.setViewText(v, text);
    }
};
```
#### 2.5 实现效果
- 新建笔记：自动记录创建时间，显示格式如 "2024-12-09 15:30"
- 修改笔记：更新时间自动更新为最新修改时间
- 列表显示：每个笔记条目下方显示对应的时间信息
- 实时更新：每次操作后时间戳自动刷新

<img width="451" height="329" alt="QQ_1765355878652" src="https://github.com/user-attachments/assets/9a006090-27df-4b6e-985c-4cce90fbd64d" />
  
关键代码文件：
- NotePadProvider.java - 数据库操作和时间戳管理
- NotesList.java - 时间戳显示和格式化
- noteslist_item.xml - 列表项布局
- NotePad.java - 常量定义

### （二）.笔记查询功能

#### 1.功能要求

点击搜索按钮，进行搜索界面。初始状态的搜索界面不显示笔记条目。在输入搜索内容或回删一部分搜索内容后，系统根据输入内容和笔记的标题进行字符串匹配，刷新符合要求的笔记显示在笔记列表上，后续如果回删搜索内容至为空后，显示所有的笔记

#### 2.实现思路和技术实现

**实现思路：**
- 界面设计：在工具栏添加搜索图标和搜索框
- 查询逻辑：使用 SQL 的 LIKE 语句进行模糊查询
- 实时搜索：监听搜索框文本变化，实时更新列表
- 结果显示：动态过滤并显示匹配的笔记

  **技术实现：**
  
#### 2.1 搜索界面设计
在 list_options_menu.xml 中添加搜索菜单项：
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <item android:id="@+id/menu_add"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_add"
          android:alphabeticShortcut='a'
          app:showAsAction="always" />
          
    <item android:id="@+id/menu_paste"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_paste"
          android:alphabeticShortcut='p'
          app:showAsAction="never" />
          
    <!-- 添加搜索菜单项 -->
    <item android:id="@+id/menu_search"
          android:title="搜索"
          android:icon="@android:drawable/ic_menu_search"
          app:showAsAction="ifRoom|collapseActionView"
          app:actionViewClass="android.widget.SearchView" />
</menu>
```    
#### 2.2 搜索逻辑实现
在 NotesList.java 中实现搜索功能的核心方法 loadNotes()：
```java
/**
 * 加载笔记列表，支持搜索过滤
 * @param query 搜索关键词，如果为null或空字符串则显示所有笔记
 */
private void loadNotes(String query) {
    String selection = null;
    String[] selectionArgs = null;
    
    // 如果有搜索关键词，构建查询条件
    if (query != null && !query.trim().isEmpty()) {
        // 同时搜索标题和内容字段
        selection = NotePad.Notes.COLUMN_NAME_TITLE + " LIKE ? OR " + 
                   NotePad.Notes.COLUMN_NAME_NOTE + " LIKE ?";
        String filterPattern = "%" + query + "%";
        selectionArgs = new String[] { filterPattern, filterPattern };
    }

    // 执行数据库查询
    Cursor cursor = managedQuery(
        getIntent().getData(),            // 内容URI
        PROJECTION,                       // 要查询的列
        selection,                        // WHERE条件
        selectionArgs,                    // WHERE参数
        NotePad.Notes.DEFAULT_SORT_ORDER  // 排序方式
    );

    // 创建适配器绑定数据
    String[] dataColumns = { 
        NotePad.Notes.COLUMN_NAME_TITLE,
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
    };

    int[] viewIDs = { 
        android.R.id.text1,
        R.id.text2
    };

    mAdapter = new SimpleCursorAdapter(
            this,
            R.layout.noteslist_item,
            cursor,
            dataColumns,
            viewIDs
    ) {
        @Override
        public void setViewText(TextView v, String text) {
            // 如果是时间戳列，格式化时间
            if (v.getId() == R.id.text2 && text != null && !text.isEmpty()) {
                try {
                    long timeMillis = Long.parseLong(text);
                    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm", Locale.getDefault());
                    text = sdf.format(new Date(timeMillis));
                } catch (NumberFormatException e) {
                    // 保持原文本
                }
            }
            super.setViewText(v, text);
        }
    };

    // 设置适配器，更新列表显示
    setListAdapter(mAdapter);
}
```
#### 2.3 实时搜索监听
在 NotesList.java 的 onCreateOptionsMenu() 方法中设置搜索监听：
```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.list_options_menu, menu);

    // 获取搜索菜单项
    MenuItem searchItem = menu.findItem(R.id.menu_search);
    SearchView searchView = (SearchView) searchItem.getActionView();
    
    if (searchView != null) {
        // 设置搜索提示文字
        searchView.setQueryHint("搜索笔记标题或内容...");
        
        // 设置搜索文本变化监听器
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                // 用户提交搜索（按回车键）
                loadNotes(query);
                // 隐藏键盘
                searchView.clearFocus();
                return true;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                // 文本变化时实时搜索
                loadNotes(newText);
                return true;
            }
        });
        
        // 设置搜索框展开/关闭监听
        searchItem.setOnActionExpandListener(new MenuItem.OnActionExpandListener() {
            @Override
            public boolean onMenuItemActionExpand(MenuItem item) {
                // 搜索框展开时
                return true;
            }

            @Override
            public boolean onMenuItemActionCollapse(MenuItem item) {
                // 搜索框关闭时，显示所有笔记
                loadNotes(null);
                return true;
            }
        });
    }

    return true;
}
```
#### 2.4 实现功能截图

(1)点击搜索按钮进行搜索界面

<img width="432" height="302" alt="QQ_1765355911291" src="https://github.com/user-attachments/assets/907182e1-dbe2-4779-935c-712a88ff65fa" />

(2)输入搜索内容，显示符合条件的笔记

<img width="422" height="405" alt="QQ_1765355921518" src="https://github.com/user-attachments/assets/162cef2e-fec9-4703-a664-c17f8fcc02d6" />

## 三.拓展附加功能

### （一）.UI美化

#### 1.功能要求

在笔记编辑时可以切换编辑界面的背景颜色，同时更换笔记列表中的该笔记颜色（和编辑界面背景颜色相同），实现统一的主题化体验。

#### 2.实现思路和技术实现

**实现思路：**
- 颜色选择机制：在编辑页面提供颜色选择菜单
- 数据存储：将颜色值保存到数据库
- 界面同步：编辑页面和列表页面显示相同颜色
- 实时反馈：选择颜色后立即看到效果

 **技术实现：**
#### 2.1 数据库扩展 - 添加颜色字段
在 NotePad.java 中定义颜色常量：
```java
public static final class Notes implements BaseColumns {
    // ... 原有字段 ...
    
    // 颜色字段和常量
    public static final String COLUMN_NAME_COLOR = "color";
    public static final int COLOR_DEFAULT = 0;    // 默认绿色
    public static final int COLOR_RED = 1;        // 红色
    public static final int COLOR_GREEN = 2;      // 绿色
    public static final int COLOR_BLUE = 3;       // 蓝色
    public static final int COLOR_YELLOW = 4;     // 黄色
    public static final int COLOR_PURPLE = 5;     // 紫色
    public static final int COLOR_ORANGE = 6;     // 橙色
}
```
####  2.2 数据库升级 - 支持颜色字段
修改 NotePadProvider.java 的 onCreate() 和 onUpgrade() 方法：
```java
// 在CREATE TABLE语句中添加颜色字段
db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
        + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
        + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
        + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
        + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
        + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
        + NotePad.Notes.COLUMN_NAME_COLOR + " INTEGER DEFAULT 0"  // 颜色字段，默认值0
        + ");");

// 在onUpgrade()中添加升级逻辑
if (oldVersion == 2 && newVersion == 3) {
    // 从版本2升级到3，添加颜色字段
    db.execSQL("ALTER TABLE " + NotePad.Notes.TABLE_NAME + 
              " ADD COLUMN " + NotePad.Notes.COLUMN_NAME_COLOR + " INTEGER DEFAULT 0");
}
```
#### 2.3 编辑页面颜色选择菜单
在 NoteEditor.java 中添加颜色选择功能：
```java
// 在onCreateOptionsMenu()中添加颜色菜单
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    super.onCreateOptionsMenu(menu);
    
    // 添加颜色选择子菜单
    SubMenu colorMenu = menu.addSubMenu("背景颜色");
    
    // 添加颜色选项
    colorMenu.add(0, NotePad.Notes.COLOR_RED, 0, "红色主题")
            .setIcon(getColorIcon(Color.RED));
    colorMenu.add(0, NotePad.Notes.COLOR_GREEN, 0, "绿色主题")
            .setIcon(getColorIcon(Color.GREEN));
    colorMenu.add(0, NotePad.Notes.COLOR_BLUE, 0, "蓝色主题")
            .setIcon(getColorIcon(Color.BLUE));
    colorMenu.add(0, NotePad.Notes.COLOR_YELLOW, 0, "黄色主题")
            .setIcon(getColorIcon(Color.YELLOW));
    colorMenu.add(0, NotePad.Notes.COLOR_PURPLE, 0, "紫色主题")
            .setIcon(getColorIcon(Color.parseColor("#9C27B0")));
    colorMenu.add(0, NotePad.Notes.COLOR_ORANGE, 0, "橙色主题")
            .setIcon(getColorIcon(Color.parseColor("#FF9800")));
    colorMenu.add(0, NotePad.Notes.COLOR_DEFAULT, 0, "默认主题")
            .setIcon(getColorIcon(Color.parseColor("#4CAF50")));
    
    return true;
}

// 创建颜色图标的方法
private Drawable getColorIcon(int color) {
    ShapeDrawable shape = new ShapeDrawable(new OvalShape());
    shape.getPaint().setColor(color);
    shape.setIntrinsicWidth(32);
    shape.setIntrinsicHeight(32);
    return shape;
}
```
#### 2.4 处理颜色选择事件
在 NoteEditor.java 的 onOptionsItemSelected() 中：

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    int colorId = item.getItemId();
    
    // 处理颜色选择
    if (colorId == NotePad.Notes.COLOR_RED || 
        colorId == NotePad.Notes.COLOR_GREEN || 
        colorId == NotePad.Notes.COLOR_BLUE || 
        colorId == NotePad.Notes.COLOR_YELLOW ||
        colorId == NotePad.Notes.COLOR_PURPLE ||
        colorId == NotePad.Notes.COLOR_ORANGE ||
        colorId == NotePad.Notes.COLOR_DEFAULT) {
        
        // 1. 保存颜色到数据库
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_COLOR, colorId);
        getContentResolver().update(mUri, values, null, null);
        
        // 2. 立即更新编辑页面背景
        updateEditorBackground(colorId);
        
        // 3. 显示提示
        Toast.makeText(this, "已应用颜色主题", Toast.LENGTH_SHORT).show();
        return true;
    }
    
    return super.onOptionsItemSelected(item);
}

// 更新编辑页面背景的方法
private void updateEditorBackground(int colorId) {
    View rootView = findViewById(android.R.id.content);
    int backgroundColor;
    
    switch (colorId) {
        case NotePad.Notes.COLOR_RED:
            backgroundColor = Color.parseColor("#FFEBEE"); // 浅红色
            break;
        case NotePad.Notes.COLOR_GREEN:
            backgroundColor = Color.parseColor("#E8F5E9"); // 浅绿色
            break;
        case NotePad.Notes.COLOR_BLUE:
            backgroundColor = Color.parseColor("#E3F2FD"); // 浅蓝色
            break;
        case NotePad.Notes.COLOR_YELLOW:
            backgroundColor = Color.parseColor("#FFFDE7"); // 浅黄色
            break;
        case NotePad.Notes.COLOR_PURPLE:
            backgroundColor = Color.parseColor("#F3E5F5"); // 浅紫色
            break;
        case NotePad.Notes.COLOR_ORANGE:
            backgroundColor = Color.parseColor("#FFF3E0"); // 浅橙色
            break;
        default:
            backgroundColor = Color.parseColor("#F5F5F5"); // 默认浅灰色
    }
    
    rootView.setBackgroundColor(backgroundColor);
    
    // 同时更新编辑框的背景，确保可读性
    EditText noteEditor = findViewById(R.id.note);
    noteEditor.setBackgroundColor(Color.TRANSPARENT);
}
```
####  2.5 列表页面显示颜色标记
修改 noteslist_item.xml 布局，添加颜色标记：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="12dp"
    android:layout_marginHorizontal="8dp"
    android:layout_marginVertical="4dp"
    android:background="@drawable/list_item_bg">

    <!-- 颜色标记条 -->
    <View
        android:id="@+id/color_indicator"
        android:layout_width="6dp"
        android:layout_height="match_parent"
        android:background="#4CAF50" />

    <!-- 内容区域 -->
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:orientation="vertical"
        android:paddingStart="12dp">

        <TextView
            android:id="@android:id/text1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="16sp"
            android:textStyle="bold"
            android:textColor="#212121" />

        <TextView
            android:id="@+id/text2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="12sp"
            android:textColor="#757575" />

    </LinearLayout>

</LinearLayout>
```
####  2.6 列表适配器绑定颜色
在 NotesList.java 的适配器中设置颜色：

```java
SimpleCursorAdapter adapter = new SimpleCursorAdapter(...) {
    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        super.bindView(view, context, cursor);
        
        // 获取颜色标记视图
        View colorIndicator = view.findViewById(R.id.color_indicator);
        
        // 从数据库读取颜色值
        int colorIndex = cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_COLOR);
        int colorValue = cursor.getInt(colorIndex);
        
        // 根据颜色值设置标记颜色
        int indicatorColor;
        switch (colorValue) {
            case NotePad.Notes.COLOR_RED:
                indicatorColor = Color.RED;
                break;
            case NotePad.Notes.COLOR_GREEN:
                indicatorColor = Color.GREEN;
                break;
            case NotePad.Notes.COLOR_BLUE:
                indicatorColor = Color.BLUE;
                break;
            case NotePad.Notes.COLOR_YELLOW:
                indicatorColor = Color.YELLOW;
                break;
            case NotePad.Notes.COLOR_PURPLE:
                indicatorColor = Color.parseColor("#9C27B0");
                break;
            case NotePad.Notes.COLOR_ORANGE:
                indicatorColor = Color.parseColor("#FF9800");
                break;
            default:
                indicatorColor = Color.parseColor("#4CAF50"); // 默认绿色
        }
        
        colorIndicator.setBackgroundColor(indicatorColor);
        
        // 可选：根据颜色设置列表项背景色
        if (colorValue != NotePad.Notes.COLOR_DEFAULT) {
            int bgColor = getLightColor(indicatorColor);
            view.setBackgroundColor(bgColor);
        }
    }
    
    // 获取浅色版本的颜色
    private int getLightColor(int color) {
        float[] hsv = new float[3];
        Color.colorToHSV(color, hsv);
        hsv[2] = 0.9f; // 提高亮度值
        return Color.HSVToColor(0x20, hsv); // 添加透明度
    }
};
```
####  2.7 创建背景资源文件
在 res/drawable/ 目录下创建 list_item_bg.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    
    <solid android:color="#FFFFFF" />
    
    <corners android:radius="8dp" />
    
    <stroke
        android:width="1dp"
        android:color="#E0E0E0" />
    
    <padding
        android:left="8dp"
        android:top="8dp"
        android:right="8dp"
        android:bottom="8dp" />

</shape>
```
创建 res/drawable/editor_backgrounds.xml 定义不同颜色的背景：
```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- 红色主题 -->
    <item android:state_selected="true" android:color="@color/red_light">
        <shape android:shape="rectangle">
            <gradient
                android:startColor="#FFEBEE"
                android:endColor="#FFCDD2"
                android:angle="135" />
        </shape>
    </item>
    
    <!-- 绿色主题 -->
    <item android:color="@color/green_light">
        <shape android:shape="rectangle">
            <gradient
                android:startColor="#E8F5E9"
                android:endColor="#C8E6C9"
                android:angle="135" />
        </shape>
    </item>
    
    <!-- 蓝色主题 -->
    <item android:color="@color/blue_light">
        <shape android:shape="rectangle">
            <gradient
                android:startColor="#E3F2FD"
                android:endColor="#BBDEFB"
                android:angle="135" />
        </shape>
    </item>
    
    <!-- 更多颜色主题... -->

</selector>
```
####  2.8 颜色资源定义
在 res/values/colors.xml 中定义颜色：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- 主题颜色 -->
    <color name="red_light">#FFEBEE</color>
    <color name="green_light">#E8F5E9</color>
    <color name="blue_light">#E3F2FD</color>
    <color name="yellow_light">#FFFDE7</color>
    <color name="purple_light">#F3E5F5</color>
    <color name="orange_light">#FFF3E0</color>
    <color name="default_light">#F5F5F5</color>
    
    <!-- 标记颜色 -->
    <color name="red_marker">#F44336</color>
    <color name="green_marker">#4CAF50</color>
    <color name="blue_marker">#2196F3</color>
    <color name="yellow_marker">#FFEB3B</color>
    <color name="purple_marker">#9C27B0</color>
    <color name="orange_marker">#FF9800</color>
</resources>
```

#### 3.实现效果界面截图
(1)在笔记编辑时对其切换编辑界面的背景颜色

- 点击笔记进入编辑界面，在编辑界面中点击右上方的改变背景颜色按钮
<img width="438" height="363" alt="QQ_1765356540840" src="https://github.com/user-attachments/assets/30f541c1-2e87-4a48-abba-6e982c13d36e" />

- 选择背景颜色
<img width="426" height="423" alt="QQ_1765356546949" src="https://github.com/user-attachments/assets/23d1b150-2c65-4447-8cd4-18c84bafc762" />

(2)更换背景颜色后的笔记列表

<img width="421" height="334" alt="QQ_1765356554692" src="https://github.com/user-attachments/assets/75338e90-b5ac-4bc1-8a2a-adb615cb6b06" />

### （二）.笔记字数统计功能

#### 1.功能要求

- 在编辑页面实时显示当前字数
- 支持统计字符总数
- 不同字数范围显示不同颜色提示
- 统计信息持久化显示

#### 2.实现思路和技术实现

**实现思路：**
- 实时监听：使用 TextWatcher 监听编辑框文本变化
- 字数计算：实时计算字符数量
- 视觉反馈：根据字数变化更新显示和颜色
- 布局集成：在编辑页面顶部添加字数统计栏

**技术实现：**
#### 2.1 修改布局文件 - 添加字数统计栏
修改 res/layout/note_editor.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#F5F5F5">

    <!-- 字数统计栏 -->
    <LinearLayout
        android:id="@+id/word_count_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:background="#4CAF50"
        android:paddingVertical="8dp"
        android:paddingHorizontal="16dp"
        android:gravity="center_vertical">

        <!-- 字数图标 -->
        <ImageView
            android:layout_width="20dp"
            android:layout_height="20dp"
            android:src="@android:drawable/ic_menu_edit"
            android:tint="#FFFFFF"
            android:layout_marginEnd="8dp" />

        <!-- 字数显示 -->
        <TextView
            android:id="@+id/word_count_text"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="字数: 0"
            android:textColor="#FFFFFF"
            android:textSize="14sp"
            android:textStyle="bold" />

        <!-- 详细统计（可选） -->
        <TextView
            android:id="@+id/char_count_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="字符: 0"
            android:textColor="#E8F5E9"
            android:textSize="12sp" />

    </LinearLayout>

    <!-- 原有的编辑区域 -->
    <view
        class="com.example.android.notepad.NoteEditor$LinedEditText"
        android:id="@+id/note"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/transparent"
        android:padding="15dp"
        android:scrollbars="vertical"
        android:fadingEdge="vertical"
        android:gravity="top"
        android:textSize="18sp"
        android:textColor="#333333"
        android:inputType="textMultiLine|textCapSentences"
        android:lineSpacingExtra="6dp"/>

</LinearLayout>
```
#### 2.2 实现字数统计核心逻辑
在 NoteEditor.java 中添加字数统计功能：

```java
// 添加成员变量
private TextView mWordCountText;
private TextView mCharCountText;
private LinearLayout mWordCountContainer;
private int mCurrentWordCount = 0;
private int mCurrentCharCount = 0;

// 在onCreate方法中初始化字数统计
private void initWordCount() {
    // 获取字数统计视图
    mWordCountText = findViewById(R.id.word_count_text);
    mCharCountText = findViewById(R.id.char_count_text);
    mWordCountContainer = findViewById(R.id.word_count_container);
    
    // 获取编辑框
    LinedEditText noteEditor = findViewById(R.id.note);
    
    // 设置文本变化监听器
    noteEditor.addTextChangedListener(new TextWatcher() {
        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {
            // 文本改变前的回调，通常不需要实现
        }

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {
            // 文本正在改变时的回调，通常不需要实现
        }

        @Override
        public void afterTextChanged(Editable s) {
            // 文本改变后的回调，这是字数统计的核心
            updateWordCount(s.toString());
        }
    });
    
    // 初始显示当前笔记的字数（如果是编辑现有笔记）
    if (mCursor != null && mCursor.moveToFirst()) {
        int colNoteIndex = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE);
        String existingText = mCursor.getString(colNoteIndex);
        if (existingText != null) {
            updateWordCount(existingText);
        }
    }
}

// 更新字数统计的核心方法
private void updateWordCount(String text) {
    // 计算字符数（包括空格和标点）
    mCurrentCharCount = text.length();
    
    // 计算字数（简单版本：按非空白字符序列计算）
    mCurrentWordCount = calculateWordCount(text);
    
    // 更新显示
    updateWordCountDisplay();
    
    // 根据字数更新颜色提示
    updateColorHint();
}

// 计算字数的方法
private int calculateWordCount(String text) {
    if (text == null || text.trim().isEmpty()) {
        return 0;
    }
    
    // 方法1：简单的按空格分割（适用于英文）
    // String[] words = text.trim().split("\\s+");
    // return words.length;
    
    // 方法2：适用于中英文混合的更准确计算
    int wordCount = 0;
    boolean inWord = false;
    
    for (int i = 0; i < text.length(); i++) {
        char c = text.charAt(i);
        
        // 如果是字母、数字、汉字，认为是单词的一部分
        if (Character.isLetterOrDigit(c) || isChineseCharacter(c)) {
            if (!inWord) {
                wordCount++;
                inWord = true;
            }
        } else {
            inWord = false;
        }
    }
    
    return wordCount;
}

// 判断是否为中文字符
private boolean isChineseCharacter(char c) {
    Character.UnicodeBlock ub = Character.UnicodeBlock.of(c);
    return ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS
            || ub == Character.UnicodeBlock.CJK_COMPATIBILITY_IDEOGRAPHS
            || ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS_EXTENSION_A
            || ub == Character.UnicodeBlock.GENERAL_PUNCTUATION
            || ub == Character.UnicodeBlock.CJK_SYMBOLS_AND_PUNCTUATION
            || ub == Character.UnicodeBlock.HALFWIDTH_AND_FULLWIDTH_FORMS;
}

// 更新显示
private void updateWordCountDisplay() {
    // 更新主字数显示
    String wordCountText = String.format("字数: %d", mCurrentWordCount);
    mWordCountText.setText(wordCountText);
    
    // 更新字符数显示
    String charCountText = String.format("字符: %d", mCurrentCharCount);
    mCharCountText.setText(charCountText);
    
    // 可以添加更多统计信息
    // 例如：汉字数、英文单词数等
}

// 根据字数更新颜色提示
private void updateColorHint() {
    int backgroundColor;
    String hintText = "";
    
    if (mCurrentCharCount == 0) {
        // 空文档 - 浅蓝色提示
        backgroundColor = Color.parseColor("#2196F3");
        hintText = "开始写作吧";
    } else if (mCurrentCharCount < 100) {
        // 短文档 - 绿色
        backgroundColor = Color.parseColor("#4CAF50");
        hintText = "短文";
    } else if (mCurrentCharCount < 500) {
        // 中等长度 - 橙色
        backgroundColor = Color.parseColor("#FF9800");
        hintText = "中等";
    } else if (mCurrentCharCount < 1000) {
        // 较长文档 - 深橙色
        backgroundColor = Color.parseColor("#FF5722");
        hintText = "较长";
    } else {
        // 长文档 - 红色
        backgroundColor = Color.parseColor("#F44336");
        hintText = "长文";
    }
    
    // 设置背景颜色
    mWordCountContainer.setBackgroundColor(backgroundColor);
    
    // 可以添加动画效果
    if (mCurrentCharCount > 0) {
        animateWordCountChange();
    }
}

// 添加动画效果（可选）
private void animateWordCountChange() {
    // 简单的缩放动画
    mWordCountText.animate()
            .scaleX(1.1f)
            .scaleY(1.1f)
            .setDuration(100)
            .withEndAction(new Runnable() {
                @Override
                public void run() {
                    mWordCountText.animate()
                            .scaleX(1.0f)
                            .scaleY(1.0f)
                            .setDuration(100)
                            .start();
                }
            })
            .start();
}
```
####  2.4 在菜单中添加统计选项
修改 res/menu/editor_options_menu.xml：

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 原有菜单项 -->
    <item android:id="@+id/menu_save"
          android:icon="@drawable/ic_menu_save"
          android:title="保存"
          android:showAsAction="ifRoom|withText" />
    
    <!-- 添加统计菜单项 -->
    <item android:id="@+id/menu_statistics"
          android:icon="@android:drawable/ic_menu_info_details"
          android:title="详细统计"
          android:showAsAction="never" />
    
    <!-- 其他菜单项 -->
</menu>
```
在 NoteEditor.java 中处理统计菜单：

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // ... 其他菜单处理 ...
    
    if (item.getItemId() == R.id.menu_statistics) {
        String text = mText.getText().toString();
        showDetailedStatistics(text);
        return true;
    }
    
    return super.onOptionsItemSelected(item);
}
```
#### 3.实现效果界面截图

<img width="547" height="360" alt="92d3b015cf4cfebb92143af0d5eecb26" src="https://github.com/user-attachments/assets/fcfc2fa8-a13b-40e4-9bae-40728e04e247" />

## 四、项目结构
```
NotePad/
├── app/
│   ├── src/main/java/com/example/android/notepad/
│   │   ├── NoteEditor.java      # 编辑页面，处理编辑逻辑
│   │   ├── NotesList.java       # 列表页面，显示笔记列表
│   │   ├── NotePadProvider.java # 数据提供者，数据库操作
│   │   └── NotePad.java         # 常量定义，数据库表结构
│   ├── src/main/res/
│   │   ├── layout/              # 布局文件
│   │   │   ├── note_editor.xml  # 编辑页面布局
│   │   │   └── noteslist_item.xml # 列表项布局
│   │   ├── menu/               # 菜单文件
│   │   ├── drawable/           # 图片和形状资源
│   │   └── values/             # 颜色、字符串等资源
│   └── build.gradle            # 项目构建配置
└── README.md                   # 项目说明文档
```
## 五、实验总结
#### 5.1 实现成果
- ✅ 完整实现了基础笔记管理功能
- ✅ 成功添加了时间戳显示功能
- ✅ 实现了实时搜索功能
- ✅ 完成了UI界面美化
- ✅ 添加了笔记颜色标记功能
- ✅ 实现了字数统计功能
 
