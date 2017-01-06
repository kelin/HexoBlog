---
title: Pawn的Relevancy（关联）Checklist分析（基于虚幻3）
date: 2016-11-26 17:41:10
categories: 虚幻3
tags:
---

在项目开发中，我发现了游戏中有一个bug。怪物会无端端消失，移动一下玩家角色，怪物又会重新出现。特别是在玩家角色前面有遮挡物的时候，很容易重现。不难猜想，就是虚幻的关联导致的这个现象，因此唯有对Pawn的关联的检查流程仔细分析。
追根溯源，最后终于定位到了如下函数：

![Pawn的关联判断函数.png](http://upload-images.jianshu.io/upload_images/3713845-7cd88fe288a20426.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个函数具体实现如下：
```
UBOOL APawn ::IsNetRelevantFor( APlayerController* RealViewer, AActor * Viewer, const FVector & SrcLocation)
{
     if ( bAlwaysRelevant )
          return TRUE ;
     if ( (NetRelevancyTime == GWorld->GetTimeSeconds ()) && (RealViewer == LastRealViewer) && (Viewer == LastViewer) )
    {
          return bCachedRelevant ;
    }
     if( IsOwnedBy (Viewer) || IsOwnedBy(RealViewer ) || this==Viewer || Viewer== Instigator
         || IsBasedOn(Viewer ) || (Viewer && Viewer->IsBasedOn (this)) || RealViewer->bReplicateAllPawns
         || ( Controller && ((Location - Viewer->Location ).SizeSquared() < AlwaysRelevantDistanceSquared )) || HasAudibleAmbientSound(SrcLocation ) )
          return CacheNetRelevancy(TRUE ,RealViewer, Viewer);
     else if ( (bHidden || bOnlyOwnerSee) && !bBlockActors )
          return CacheNetRelevancy(FALSE ,RealViewer, Viewer);
     else if ( Base && ( BaseSkelComponent || ((Base == Owner) && !bOnlyOwnerSee )) )
          return Base ->IsNetRelevantFor( RealViewer, Viewer, SrcLocation );
     else
    {
#ifdef USE_DISTANCE_FOG_OCCLUSION
          // check distance fog
          if ( RealViewer->BeyondFogDistance (SrcLocation, Location) )
              return CacheNetRelevancy(false ,RealViewer, Viewer);
#endif
          // check against BSP - check head and center
          //debugf(TEXT("Check relevance of %s"),*(PlayerReplicationInfo->PlayerName));
          FCheckResult Hit (1.f);
          if ( !GWorld ->SingleLineCheck( Hit, this , Location + FVector (0.f,0.f,BaseEyeHeight), SrcLocation, TRACE_World|TRACE_StopAtAnyHit |TRACE_ComplexCollision, FVector(0.f,0.f,0.f) )
             && ! GWorld->SingleLineCheck ( Hit, this, Location, SrcLocation , TRACE_World|TRACE_StopAtAnyHit |TRACE_ComplexCollision, FVector(0.f,0.f,0.f) )
             && !IsRelevantThroughPortals (RealViewer) )
         {
              return CacheNetRelevancy(FALSE ,RealViewer, Viewer);
         }
          return CacheNetRelevancy(TRUE ,RealViewer, Viewer);
    }
}
```
可以看到，Pawn的关联判断流程：
1、先判断是否是bAlwaysRelevant，如果是，则直接返回true
2、判断是否已经检查过了，是则直接返回上一次检查结果
3、判断以下条件，有一个成立则认为是关联：
           是否被Viewer拥有，
           是否被RealViewer拥有，是否viewer就是自己本身，
           viewer是否是本身的Instigator（注意，Instigator 和 owner不要混淆了哦），
           是否base on viewer或者是viewer base on自己，
           是否RealViewer的bReplicateAllPawns为true，
           自己和viewer是否在一定会关联的距离内，
           是否会听到目标位置的声音
4、判断 ( (bHidden || bOnlyOwnerSee) && !bBlockActors )，不能被碰撞，而且被隐藏了或者自由owner能看见。
5、判断是否有base，有的话检查base是否是关联的，如果base是关联，则这个pawn也是关联的。
6、以下条件任何一个成立，则返回不关联
            通过线性检测，检测到这个Pawn的眼睛的位置和SrcLocation之间有遮挡（SrcLocation 可以理解为Viewer的location，当然为了更好的体验感，其实还增加了预判，具体的实现在FNetViewer::FNetViewer()函数中）；
            这个Pawn的位置和SrcLocation之间有遮挡；
            不能通过某个房间的入口看得到RealViewer（通过遍历检查RealViewer::VisiblePortals的成员实现。TickSpecial()会不断调用CheckPortalVisible()更新Controller中的VisiblePortals）；

问题就出在第6个条件那里。
这个Pawn就是要判断是否关联的怪物，而第六个条件只是判断了：怪物的眼睛到玩家的位置做射线检测、怪物的位置到玩家的位置做射线检测 ，但是，玩家的眼睛的位置到怪物的位置这个射线检测没有判断，因此如果怪物检测不到玩家，而玩家却刚好能看得到怪物的时候，就会出现这个怪物不关联的bug了。
知道问题，改起来也简单，增加一个射线检测就行了。上面的函数改为下面这样，就OK了。

```
UBOOL APawn :: IsNetRelevantFor( APlayerController * RealViewer , AActor * Viewer, const FVector & SrcLocation )
{
     if ( bAlwaysRelevant )
          return TRUE ;
     if ( (NetRelevancyTime == GWorld ->GetTimeSeconds ()) && ( RealViewer == LastRealViewer ) && (Viewer == LastViewer) )
    {
          return bCachedRelevant ;
    }
     if ( IsOwnedBy ( Viewer) || IsOwnedBy (RealViewer ) || this ==Viewer || Viewer== Instigator
         || IsBasedOn (Viewer ) || ( Viewer && Viewer ->IsBasedOn ( this)) || RealViewer ->bReplicateAllPawns

         || ( Controller && ((Location - Viewer ->Location ). SizeSquared() < AlwaysRelevantDistanceSquared )) || HasAudibleAmbientSound (SrcLocation ) )
          return CacheNetRelevancy (TRUE , RealViewer, Viewer );
     else if ( ( bHidden || bOnlyOwnerSee ) && !bBlockActors )
          return CacheNetRelevancy (FALSE , RealViewer, Viewer );
     else if ( Base && ( BaseSkelComponent || ((Base == Owner ) && !bOnlyOwnerSee )) )
          return Base -> IsNetRelevantFor( RealViewer , Viewer , SrcLocation );
     else
    {
#ifdef USE_DISTANCE_FOG_OCCLUSION
          // check distance fog
          if ( RealViewer ->BeyondFogDistance ( SrcLocation, Location ) )
              return CacheNetRelevancy (false , RealViewer, Viewer );
#endif
          // check against BSP - check head and center
          //debugf(TEXT("Check relevance of %s"),*(PlayerReplicationInfo->PlayerName));
          FCheckResult Hit (1.f);
          if ( !GWorld -> SingleLineCheck( Hit , this , Location + FVector (0.f,0.f, BaseEyeHeight), SrcLocation , TRACE_World |TRACE_StopAtAnyHit | TRACE_ComplexCollision, FVector (0.f,0.f,0.f) )
             && ! GWorld ->SingleLineCheck ( Hit, this , Location , SrcLocation , TRACE_World |TRACE_StopAtAnyHit | TRACE_ComplexCollision, FVector (0.f,0.f,0.f) )
              && !IsRelevantThroughPortals ( RealViewer) )
         {
              UBOOL RealViewerCanSeeThis = TRUE;
              APawn * RealViewerPawn = RealViewer ->Pawn ;
              if (NULL != RealViewerPawn)
             {
                  RealViewerCanSeeThis = GWorld ->SingleLineCheck ( Hit, this , RealViewerPawn ->Location + FVector (0.f,0.f,RealViewerPawn -> BaseEyeHeight), Location , TRACE_World |TRACE_StopAtAnyHit | TRACE_ComplexCollision, FVector (0.f,0.f,0.f));
             }
              if (RealViewerCanSeeThis )
                  return CacheNetRelevancy (TRUE , RealViewer, Viewer );
              else
                  return CacheNetRelevancy (FALSE , RealViewer, Viewer );
         }
          return CacheNetRelevancy (TRUE , RealViewer, Viewer );
    }
}
```