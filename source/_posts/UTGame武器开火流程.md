---
title: UTGame武器开火流程
date: 2016-11-26 17:36:34
categories: 虚幻3
tags:
---

| 执行的对象 | 本地客户端 | Server | 其他玩家客户端 |
|:-----------| :----------- | :----------- | :-----------:|
| playerController| StartFire() | ||
| UTPawn | StartFire()| ||
| Weapon(LinkGun) |StartFire()<br>StartFire主要做了两件事：<br>1. 本地直接执行BeginFire()<br>2. 如果是网络游戏会远程调用ServerStartFire()      | ||
| Weapon(LinkGun) | BeginFire()<br>注意，此时Weapon的状态是Active，执行的是状态机里面的BeginFire()函数。<br>这里主要做了两件事：<br>1. 调用Global.BeginFire()。<br>这个函数调用了SetPendingFire()，而转而调用了InventoryManager的SetPendingFire()，这个函数只有一行代码：PendingFire[InFiringMode] = 1;将这个值设为1，主要是用来记录开火模式，鼠标左右键有不同开火模式。<br>2. 如果能开火，调用SendToFiringState()。<br>这个函数设置了开火模式（UTPawn的SetFiringMode()函数），然后跳转到开火状态WeaponFiring。<br>3. UTWeapon重写了这个BeginFire，并加入了checkRoom()功能，主要用在狙击枪开镜，这里不展开了，其实就是修改FOV视角。|ServerStartFire()<br>主要是做了一件事：<br>执行BeginFire()。与本地客户端相同。||
| Weapon(LinkGun) | 进入WeaponFiring状态，执行BeginState()。<br>这个函数做了两件事：<br>1. FireAmmunition()<br>这个函数也是干了两件事<br>（1)ConsumeAmmo()<br>消耗子弹。<br>（2）根据子弹类型调用不同逻辑，在这里以EWFT_Projectile为例，则执行ProjectileFire()。<br>ProjectileFire()主要做两件事：<br>  IncrementFlashCount()<br>     这个函数调用了UTPawn的IncrementFlashCount()函数。后面两行继续详细分析。<br> - 因为是客户端，所以不会执行这件事。（Spawn子弹）。<br>2. TimeWeaponFiring()<br>这个函数是两次开枪时间间隔相关的计时。| 1. 这里与本地客户端唯一区别就是ProjectileFire()函数，服务器还会做一件事情就是：创建子弹。<br>2. 创建子弹流程如下：<br>a) 调用Instigator的GetWeaponStartTraceLocation函数。如果Instigator有controller，这函数会返回controller的location，不然就返回它自己眼睛的位置，StartTrace。<br>b)    获得发射方向。GetAdjustedAim函数是一系列让weapon, the pawn and the controller调整子弹发射方向的处理的开始。它调用 了Pawn(如果有)的GetAdjustedAimFor()，然后调用AddSpread()增加散射调整后return最终的角度。Pawn的GetAdjustedAimFor()。<br>如果Pawn没有controller，则返回GetBaseAimRotation()的计算结果，否则调用controller的GetAdjustedAimFor()，让controller调整角度。<br>c)   调用GetPhysicalFireStartLoc()获得子弹的创建位置（一般是枪口位置）<br>d) 如果子弹创建位置和StartTrace不相等，那么需要根据用StartTrace、发射方向、射程来做射线检测，预测子弹的碰撞，CalcWeaponFire()，据碰撞点和子弹创建位置求得新的发射方向。<br>e)  Spawn子弹并init。||
| UTPawn | IncrementFlashCount()<br>这个函数修改了FlashCount，再次设置SetFiringMode()保证开火模式正确，然后执行FlashCountUpdated()函数，FlashCountUpdated()函数会判断FlashCount，如果FlashCount>0，那么执行WeaponFired()，否则WeaponStoppedFiring()。|逻辑和本地客户端一样。<br>此外，因为FlashCount这个值是repnotify修饰的，所以会复制到其他客户端。||
|UTPawn |WeaponFired()<br>这个函数主要是利用WeaponAttachment播放开火特效、角色动画、音效等。| |FlashCount的改变被复制下来，并执行ReplicatedEvent，这个函数内这时会根据参数名调用FlashCountUpdated函数。|
|UTPawn | | | WeaponFired()函数 |

上表描述便是主要的UTGame开火流程。
PS：关于子弹Projectile：
Projectile的物理相关属性中，bCollideWord被设为true（表示物体是否与关卡几何（CSG）发生collide。），因此，击中关卡中的物体的时候，会调用HitWall事件，进而调用TakeDamage()函数。

居然吐着血，耐着markdown难用的表格写下来了。。。