---
title: 封装内嵌UICollectionView和UIPageControl的ScrollView
date: 2020-09-21 00:38:36
tags:
---

在需求中涉及到一个比较通用的控件，ScrollView里面嵌入CollectionView，封装一下，后面再有相同交互不用重复造轮子。

##### 一。交互样式

控件交互：
![](http://km.oa.com/files/photos/pictures/201707/1501504788_13_w752_h510.jpg)


如下类似样式都可以复用同一控件:
![](http://km.oa.com/files/photos/pictures/201707/1501504866_52_w740_h416.jpg)
![](http://km.oa.com/files/photos/pictures/201707/1501504879_57_w1468_h912.jpg)

##### 二。接口

* 接口

init的时候传入view布局相关的TBCollectionViewParamsModel参数；拿到数据后调用setDataList传入数据，展示CollectionScrollView。

```
@interface TBHorizontalItemListView : UIView

- (instancetype)initWithFrame:(CGRect)frame collectionViewParamsModel:(TBCollectionViewParamsModel *)viewParams;

- (void)setDataList:(NSArray<TBCollectionDataListModel *>*)listData;

@end    

```

* 参数

```
@interface TBCollectionViewParamsModel : NSObject
@property (nonatomic, assign) CGSize itemSize;                  //collectionView的cell大小
@property (nonatomic, assign) CGFloat minimumInteritemSpacing;  //collectionView的cell间水平间距
@property (nonatomic, assign) CGFloat minimumLineSpacing;       //collectionView的cell间的竖直间距
@end


@interface TBCollectionDataListModel : NSObject
@property (nonatomic, retain) NSArray<id> *itemList;            //单个collectionView中的数据list
@property (nonatomic, strong) Class cellClass;                  //单个collectionView中使用的cell类型, 需要实现TBCollectionViewCellProtocol
@property (nonatomic, assign) int type;                         //扩展，暂时无用
@end

```

##### 三。实现

![](http://km.oa.com/files/photos/pictures/201707/1501504913_57_w718_h556.jpg)UICollectionViewUICollectionViewUICollectionViewUICollectionView


* 灰色的是容器`View`
* 紫色的是`UIScrollView`
* 蓝色的是`UICollectionView`
* 红色的是`UICollectionViewCell`
* 下方小点点是`TBScrollPageControl`

关键代码：

根据setDataList传入的数据创建CollectionView并为其布局

```
- (void)initCollectionViews
{
    _bgScrollView.contentSize = CGSizeMake(TBHorizontalItemListViewWidth * _listData.count, 0);
    
    CGFloat x_offset = 0;
    for (NSInteger i = 0; i < _listData.count; i++)
    {
        UICollectionViewFlowLayout *flowLayout = [self getCollectionViewFlowLayout:_viewParams];
        
        CGRect frame = CGRectMake(x_offset + 23 / 2, 20, TBHorizontalItemListViewWidth - 23, 199);
        UICollectionView *collectionView = [[UICollectionView alloc] initWithFrame:frame collectionViewLayout:flowLayout];
        collectionView.tag = 100+i;
        collectionView.dataSource = self;
        collectionView.delegate = self;
        collectionView.alwaysBounceHorizontal = NO;
        collectionView.alwaysBounceVertical = YES;
        collectionView.backgroundColor = [UIColor colorWithWhite:0 alpha:0];
        collectionView.showsHorizontalScrollIndicator = NO;
        collectionView.showsVerticalScrollIndicator = NO;
        collectionView.scrollEnabled = NO;
        collectionView.backgroundColor = [UIColor blueColor];
        [_bgScrollView addSubview:collectionView];
        
        Class cellClass = [_listData objectAtIndex:i].cellClass;
        NSString *identifier = [NSString stringWithFormat:@"ItemLandscapeCollectionCellIdentifier_%ld",(long)collectionView.tag];
        [collectionView registerClass:cellClass forCellWithReuseIdentifier:identifier];
        
        x_offset += TBHorizontalItemListViewWidth;
        [collectionView reloadData];
    }
    [self layoutIfNeeded];
}

```

CollectionView的代理：

```
#pragma mark - UICollectionDataSource

- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
    return _viewParams.itemSize;
}

- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    NSInteger groupIndex = collectionView.tag - 100;
    return _listData[groupIndex].itemList.count;
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    /**取数据*/
    NSInteger groupIndex = collectionView.tag - 100;
    TBCollectionDataListModel *listModel = _listData[groupIndex];
    id itemModel = listModel.itemList[indexPath.row];
    
    /**创建cell&&赋值*/
    NSString *identifier = [NSString stringWithFormat:@"ItemLandscapeCollectionCellIdentifier_%ld",collectionView.tag];
    UICollectionViewCell<TBCollectionViewCellProtocol> *cell = [collectionView dequeueReusableCellWithReuseIdentifier:identifier forIndexPath:indexPath];
    if ([cell respondsToSelector:@selector(setModel:)])
    {
        [cell setModel:itemModel];
    }
    
    return cell;
}

```
