# 毕设
毕设（第一次用这个放，玩玩）
md咋写来着？忘了，先随便写点，毕业设计pygame的游戏设计，存着玩玩

# 亮点分析

1. 游戏风格：这个游戏采用了像素化的风格，并带有很多霓虹灯效果和动画。这种设计风格非常流行，可以使游戏看起来非常酷。

2. 对象组织：在游戏中，有很多不同类型的对象（例如玩家、树、苹果），并且它们都需要以某种方式组织起来。作者使用了 Pygame 的精灵组织功能，这样可以很容易地将所有相关对象分为一组，并在更新时一起处理。

3. 碰撞检测：当游戏中的两个对象相互作用时，通常需要检测它们是否发生了碰撞。Pygame 提供了方便的碰撞检测 API，使得编写碰撞代码变得很容易。

4. 动画：在游戏中，有很多对象都需要动画效果，比如水的波动、树的摇晃等等。作者使用了 Pygame 的 Surface 类来创建帧，并使用定时器调整每一帧之间的时间差，从而制造出流畅的动画效果。

5. 粒子特效：在游戏中，有很多需要特殊效果的场景，比如树被砍倒时，需要飞溅出木屑和树枝。作者使用 Pygame 的蒙版表面和精灵组织功能创建了一些简单的粒子特效，这使得游戏看起来更加真实。


# 写点记录
## 3.15 
碰撞的补全，今天是视频P11 碰撞的内容  
首先改错，上次的发生碰撞后精灵向后退的原因是sprite错写成了self，至于为什么没搞懂，可以整理后当作毕业答辩的论述，需要写进论文  
然后上下的碰撞未实现是以为我没有写，所以当时看论文才会只有left和right，这并不是左右代替了上下进行碰撞判断，往后还需仔细阅读代码（菜就是菜，xiba）  
补齐了上下的碰撞判断后有一句 `self.collision('horizontal')` 需要搞懂为什么，有思路后补全  
第三是修改了精灵的初始位置，在level中的setup的player中，直接调用tmx中设置好的初始位置  
最后是设置了边界和高坡，地图中有一个Collision，后面有一段改完后会直接将边界显示，那一段为什么需要搞清楚  
就这么多  
#### 换行是空格，md的格式符后要空格

## 3.17 3.18 P12
苹果的创建和采摘，树木砍伐的判定，工具使用的判定 今天是视频P12的内容
创造苹果开始，先导入，settings中有`APPLE_POS`可以确定苹果的生成位置,  
建立果实的位置，先完成一个for循环，建立一个随机数,进行苹果的随机生成,调用class类的Generic,有四个变量,pos\surf\group还有一个z，surf很简单，就是`surf = self.apple_surf`直接赋值,  
而group是？`groups=[self.apple_sprites,self.groups()[0]]`,  
z直接引用贴图的设置`z = LAYERS['fruit']`，pos是两个值x和y，`x = pos[0] + self.rect.left`意思就是从窗口最左边到树的距离，y同理。最后调用create_frult就会生成苹果  
树的属性(tree attributes)，先给一个健康值health，再给一个alive，对树活着的判断并赋值为true，因为树的粗细的不同的所以调用时应该做一个判断`f'../graphics/stumps/{"small" if name == "Small" else "large"}.png'`然后在转换，转换？？？  
还需要一个`invul_timer`？？？  
另一个方法damage，健康值-1，还要移除一个苹果，如果数量为0则没有意义，所以先做一个判断苹果的数量是大于零的，`random_apple = choice(self.apple_sprites.sprites()) random_apple.kill()`？？？解惑  
这个方法将会发生在level中，调用方法先，在setup中的  
`for obj in tmx_data.get_layer_by_name('Trees') :
    Tree((obj.x,obj.y),obj.image,[self.all_sprites,self.collision_sprites,self.tree_sprites],obj.name)` 要保证self.all_sprites为第一项，否则`groups=[self.apple_sprites,self.groups()[0]],`会无法运行。然后在#player中假如参数`trees = self.tree_sprites`,然后再player的_init_方法中加入默认参数tree_sprites,然后在尾部加入#interaction`self.tree_sprites = tree_sprites`，然后再use_tools我们加入判断三种工具的使用，因为是砍伐，所以暂时只做斧头axe的，为此先准备定位，定义方法`def get_target_pos(self)`settings中有工具的定位，直接调用即可，记得更新
，继续判断工具的使用 
`if self.selected_tool == 'axe' :
     for tree in self.tree_sprites.sprites() :
        if tree.rect.collidepoint(self.target_pos) :
            tree.damge()`
最后得到树木健康-1的方法调用，结束这一段，但到此结束的话会出现一个问题，未使用工具也会导致苹果的收集，所以再timer中update函数需要修改 
```
def update(self) :
    current_time = pygame.time.get_ticks()
    if current_time - self.start_time >= self.duration :
        # self.deactivate() 如果在这里，每次运行后都会归零，下面的判断不会生效
        if self.func and self.start_time != 0 :
            self.func()  
        self.deactivate()   # 开始判断
```
这里先解释第二个if判断语句，`and self.start_time != 0`？？？  
然后一开始deactivate是再第二个if语句上方的。。。。（👆原因有写）更改后，程序正常，开始增加砍树的部分，定义函数check_death,判断引用，当_init_函数中的health经过damge函数的调用导致health<=0时，树木死亡条件成立，记得调用更新，alive判断未false，？？？
'''
def check_death(self) :
    if self.health <= 0 :
        # print('dead')
        self.image = self.stump_surf
        self.rect = self.image.get_rect(midbottom = self.rect.midbottom)
        self.hitbox = self.rect.copy().inflate(-10,-self.rect.height * 0.6) # w,h
        self.alive = False
'''  
？？？代表没写完或者有疑问，后续论文补充，十八号任务完成，md的代码块和插入图片需要学习一下，giao

## 3.25 P13
苹果树消失的粒子特效,整体可写的不多,只要把参数全部搞明白写上就行，这里不过多赘述，马上上课了

## 3.27 P14
玩家库存
建立字典进行物品的存储 `self.item_inventory` ,需要链接player和sprites中的Tree类,必须在level中完成链接,
在level中添加player_add类,获取 `self.player.item_inventory[item]` 数值是+= 1,会更加方便些,这意味着在setup中需要
更新创建tree,添加的 `player_add = self.player_add` 并且确保只在内部调用,在sprites中的tree类中的__init__添加
player_add参数，并在内部调用,当树被破坏时，将会 `self.player_add('apple')` 这将完成苹果存储的操作，树的操作同理
在 `def check_death(self)` 中调用 `self.player_add('wood')` ,在树木收集完成。

## 3.28 p15
新的一天开始  
使用用tiled中player图层中的Bed选项,将这一块区域设置为“睡觉”的判定区域，当玩家控制sprites移动到该区域并且按回车时，判定
玩家将执行睡觉并进入新的一天的操作,在sprites中创建一个新的类为instruction,类的属性继承Generic,初始化对象的属性中有pos, size, groups, name
这里不需要surface，因为无论如何，这种sprites永不可见, `surf = pygame.Surface(size)` ,现在要继承多个父类方法所以调用
super方法 `super().__init__(pos, surf, groups)` ，现在在level中的播放器设置，判断玩家是否是在'Bed'区域中，导入sprites中的Interaction并且赋参数,  
`
if obj.name == 'Bed' :  
    Interaction(pos = (obj.x,obj.y),  
                size = (obj.width,obj.height),  
                groups = self.interaction_sprites,  
                name = 'Bed' # obj.name  
                )
`
name用'Bed'或者obj.name都可以，现在将 `interaction = self.interaction_sprites` 加入循环中的判定，并在player的内部
加入参数interaction，现在就检查玩家是否在判定的区域内并按下了回车，现在判断输入回车时，玩家是否Bed sprite相重叠
`collided_interaction_sprites = pygame.sprite.spritecollide(self,self.interaction,False)` 
三个值分别是sprite, group, dokill，确定玩家的状态后，开始重启关卡的工作，重启关卡意味着所有树会得到新的苹果，如果还活着的树
需要重置整个level，tree调用完成的函数create_fruit，再使用for循环进行消除现有的苹果的操作，这基本上是一个重置，现在需要
玩家重置和画面过渡一起运行，现在player中定义sleep的状态为False，如果玩家按下回车，设置为True，玩家是否进行sleep操作的判断
就这样，现在开始过渡，创建过渡用的类transition，定义常用的setup设置，创建覆盖层，设置宽度和高度，即窗口的大小获取，color为255
设置图像填充，导入到level中，现在图像会从全白快速到黑色，这是正确的，每一帧的颜色都是变得越来越暗，加入参数 `special_flags = pygame.BLEND_RGBA_MULT`
会使变化更好的过渡，但因为self.color运算后会为负数，所以会导致程序崩溃，需要加上判断speed要乘以负一，color超过二五五要重新赋值
现在还有三件事，重置方法，唤醒层，设置速度为-2在结束过渡时，
在睡觉时移动是奇怪的，需要加上睡觉时移动要赋值为False

## 3.29 p16
耕种的土壤
导入sprites方便管理，pygame.image.load导入土壤的图soil_surf，这里有三个问题需要注意，首先判断该地区是否适合耕种，其次地区是否超过边界，最后判断土壤里是否有其他植物，所以要求出全部的可耕种那个瓷砖地格，在map.tmx中的Farmable是可耕种位置,求出地图的高和宽共有多少块瓷砖，tiled中的地图属性也有，创建列表使每一块瓷砖都在其中，标记好可耕种的瓷砖地块后，创建新方法create_hit_rects和新的列表，对存储的地块循环，先行后列，再进行枚举,进行判断如果地块再列表中的位置是有'F'表示的，那么输出这个地块的列表位置，x的位置是列表位置乘以瓷砖大小，y的位置同理，最后将所有的值赋给空的列表中，申请新的函数get_hit，创建工具的碰撞点进行判断，获取x和y，x = rect.x // TILE_SIZE和y = rect.y // TILE_SIZE以此来获取平铺的位置，检查列表中是否有'F',先获取列，再到行中进行判断，如果含有'F'，则再列表中append('X')，这个方法会通过SoilLayer前往player发生再level内部的setup中的Player中soil_layer,再player中加上属性soil_layer,最后在interaction中把参数变为实际属性，现状使用工具时，调用soil_layer.get_hit(self.target_pos),现在前面赋值x的地方创建一个土壤瓦片SoilTile，这样就完成了耕地的创建

## 3.29 P18 p17未作
浇水的逻辑
与耕种的土壤逻辑相似，首先再use_tool中将soil_layer写入水壶的判断中，给予参数self.target_pos，再sioil中创建新的方法
water用来获取浇水的位置，有两件事要做在这里，首先是将列表中的耕地表加上'w'进行标记，其次是定义一个WaterTile的类表示这块耕地上有水，再类中有三件事要做，首先是复制土壤耕地的位置意味着水瓦的位置应该和耕地的位置相同，其次是导入图像的路径，最后
重新开始的一天，水会消失，消失的水砖定义为remove_water，检视Self.grid中的'W'并且删除，最后再level中的reset中调用方法
完成水砖的消除

## 4.7 p20 p21 p19
创建作物  
收获  
雨

## 4.8 p22
白天到黑夜