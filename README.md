# 毕设
毕设（第一次用这个放，玩玩）
md咋写来着？忘了，先随便写点，毕业设计pygame的游戏设计，存着玩玩

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
