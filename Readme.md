# Word Battle 单词消除游戏系统

## **项目介绍**

**单词消除游戏总体介绍：** 

单词消除游戏由两类参与者组成，闯关者（即游戏玩家），出题者（为游戏增加游戏中使用单词）。游戏规则为，游戏每一轮，程序会根据该关卡难度，显示一个单词，一定时间后单词消失。闯关者需要在相应地方输入刚刚显示并消失的单词，如果闯关者输入正确（即闯关者输入的单词与刚刚显示的单词完全一致，包含大小写）则为通过。一关可以由一轮或者多轮组成。

### 阶段一

闯关者属性要求必须含有：闯关者姓名、已闯关关卡数、闯关者经验值、闯关者等级，若有需要可以自行添加其他属性。出题者属性要求必须含有：出题者姓名、出题者出题数目，等级，若有需要可以自行添加其余属性。

**具体功能**包括：

- 实现闯关者，出题者本地的注册，登录。
- 程序支持多人注册，同一时间只有一人可以登录。
- 任何角色均可查询所有闯关者，出题者，按照属性查找相应闯关者，出题者。
- 可以根据闯关者闯过关卡数，经验，等级等对闯关者排名，根据出题者出题数目，等级对出题者排名。

**要求**：

- 采用面向对象的方式，使用类设计。
- 在设计类时请注意类的继承关系，关注闯关者，出题者的关联（闯关者与出题者有共同的基类）。
- 使用文件或者数据库作为存储对象，自行设计设计文件格式，或者数据库表结构。
- 在题目的要求下自行扩展功能，功能扩展适当者酌情加分。

### 阶段二

闯关者即为游戏玩家，已经注册并登录的玩家可以在系统进行单词消除游戏。每一关的难度要有所增加，体现为如下三个条件中的一个或者多个：1、单词难度可以递增或者持平（即长度加长或不变）； 2、进行轮数增多（即单词数目如：前三关仅仅通过一个单词就过关，后续需要通过两个，三个甚至更多才过关）； 3、单词显示时间缩短（随着关卡的增加显示时间越来越短）。

闯关者每闯过一关，增加一定经验值。经验值会根据闯过的该关卡的关卡号、该关的闯关耗费时间共同决定。当经验值累计到一定程度闯关者等级增加。闯关失败需要重新闯该关。

游戏自带词库，而且已经注册的出题者可以为系统出题，即增加词库的新词，已经存在的单词不能再次添加（词库中的单词构成一个单词池，但建议根据单词的长度来组织存储，每次出题时，系统从该单词池中按照关卡难度随机的选择相应长度的单词）。每成功出题一次，更新该出题者的出题数目。出题者等级根据出题人成功出题数目来升级。

- 必须在题目一的基础上进行修改。
- 请根据要求设计每一关的出题方式，注意随着关卡数增加，题目难度增加。请合理处理出题人新添加新词的使用方式，并且新加词组不会影响游戏难度。
- 设计闯关者经验值，等级增加策略。出题者等级升级策略。



## 项目设计

本项目开发分为三个阶段，串行实现，每一个阶段的项目都基于前一个项目，故在git中会分别用三个 Branch 保存。

基于 C++/Qt 开发，总体设计基于C/S模型，在设计将实现划分为服务端和客户端，便于开发和维护。程序的主要模块包括：

- 游戏界面交互-C端
- 用户数据维护-S端
- 游戏词库维护-S端

部分游戏设计思路参考 Qt Documentation 中的 [Tetrix Example](https://doc.qt.io/qt-5/qtwidgets-widgets-tetrix-example.html)。

### 游戏界面交互模块

**程序功能划分：**

| 功能名称       | 开发阶段 | 备注                   |
| -------------- | -------- | ---------------------- |
| 界面控制       | 1        |                        |
| 注册/登录界面  | 1        |                        |
| 出题界面和逻辑 | 2        | `BattleBoard` 作为基类 |
| 闯关界面和逻辑 | 2        | `BattleBoard` 作为基类 |
| 排行榜界面     | 1        |                        |



游戏的**注册/登录界面**应该是最先显示出来的界面。对应的 `AccountWindow` 类应该包含以下功能：

- 与用户数据模块交互，判断用户名是否存在。
- 读取用户输入信息，并转换为用户数据存储格式。
- 与 `UserControl` 交互，更新游戏程序的当前使用用户。
- 根据某种依据，保存用户密码的时候应先在本地做**数据加密计算**。add

**排行榜界面**对应 `RankWindow` 类：

- 对给定的用户信息进行显示，根据用户类型和输入的用户数量决定显示的位置。
- 读取数据库信息，将用户转为上面显示方法需要的特定格式。同时除去默认方式外，允许指定某种属性进行排序，调整升序降序等。add
- 查询用户并显示。

> This example shows how a simple game can be created using only three classes:
>
> - The `TetrixWindow` class is used to display the player's score, number of lives, and information about the next piece to appear.
> - The `TetrixBoard` class contains the game logic, handles keyboard input, and displays the pieces on the playing area.
> - The `TetrixPiece` class contains information about each piece.

仿照这里的设计思路，我的**闯关/出题界面**部分也分为类似的 Classes：

- `BattleBoard` 作为基类，有如下的功能
  - 用户相关信息的显示，规定格式与区域
  - 当前单词的显隐控制
  - 提供面向用户的单词输入文本框
  - 界面跳转按钮
- 游戏逻辑控制
  - 当前关卡信息，当前词库信息显示
  - 词汇显示

- `VocabularyBoard` 作为 `BattleBoard` 的继承类，额外增加：
  - 当前出题者信息
  - 判断当前单词
- `ChallengeBoard` 作为 `BattleBoard` 的继承类，额外增加：
  - 将当前闯关者的信息转换为基类显示需要的格式后显示
  - 游戏逻辑部分。根据关卡计算单词消失时间

### 用户数据维护模块

程序功能划分：

| 功能名称             | 开发阶段 | 备注 |
| -------------------- | -------- | ---- |
| 用户数据的存储       | 1        | S端  |
| 请求更新用户数据信息 | 1        | C端  |
|                      |          |      |

 `UserControl` 类来控制

- **当前**用户信息的临时存储，以及更新保存。

`ClientAccess` 类在**C端**负责所有C端与S端的通讯。

- 更新用户信息
- 增加用户
- 登录匹配

`UserInfo` 类是存储用户信息的基类，包含：

- 用户名
- 更新等级、经验值、游戏时间的方法
- 



`ServerAccess` 类在**S端**负责所有C端与S端的通讯。

### 游戏词库维护模块

| 功能名称 | 开发阶段 | 备注 |
| -------- | -------- | ---- |
|          |          |      |
|          |          |      |
|          |          |      |