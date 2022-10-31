## 🌈CopyBox

####  主要内容

---

copybox是一个集成的项目，用于练手一些功能，做成一个可以按需安装模块的应用

以下是前期构想：

1. - [ ] 集成BaseActivity
2. - [ ] 可以体验一下Android原生的theme适配，可以动态更改主题
3. - [ ] 可以搭配一些硬件进行使用，例如nfc等
4. - [ ] 添加用户登录绑定这一系列逻辑
5. - [ ] 集成AndroidX
6. - [ ] 粘贴板功能
7. - [ ]  集成自己的开源输入法，不希望自己的输入记录掌握在别人手中
8. - [ ] 一个聊天系统，可以上手websocket
9. - [ ] 能上架应用商店
10. - [ ] 电子宠物，一些后台操作可以交给他们来进行
11. - [ ] 可以做一些权限控制类的功能
11. - [ ] android要做模块化
12. - [ ] 看有啥新的黑科技再加进去

以上芜湖～

---

## 06-24

- [ ] 适配原生的theme样式

  有一个官方给的样式[预览网站](https://material.io/resources/color/#!/?view.left=0&view.right=0&primary.color=FF8A80),可以通过定义各种值来查看页面的效果

  light：

  ```xml
  <style name="Theme.CopyBox" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
          <!-- Primary brand color. -->
          <item name="colorPrimary">@color/purple_500</item>
          <item name="colorPrimaryVariant">@color/purple_700</item>
          <item name="colorOnPrimary">@color/white</item>
          <!-- Secondary brand color. -->
          <item name="colorSecondary">@color/teal_200</item>
          <item name="colorSecondaryVariant">@color/teal_700</item>
          <item name="colorOnSecondary">@color/black</item>
          <!-- Status bar color. -->
          <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
          <!-- Customize your theme here. -->
      </style>
  ```

  Night :

  ```xml
  <style name="Theme.CopyBox" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
          <!-- Primary brand color. -->
          <item name="colorPrimary">@color/purple_200</item>
          <item name="colorPrimaryVariant">@color/purple_700</item>
          <item name="colorOnPrimary">@color/black</item>
          <!-- Secondary brand color. -->
          <item name="colorSecondary">@color/teal_200</item>
          <item name="colorSecondaryVariant">@color/teal_200</item>
          <item name="colorOnSecondary">@color/black</item>
          <!-- Status bar color. -->
          <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
          <!-- Customize your theme here. -->
      </style>
  ```

  

- [x] 通过navigation控制跳转等操作