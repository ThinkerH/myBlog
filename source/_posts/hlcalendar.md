---
layout: post
title: iOS HLCalendar-日历控件
tags: 开发
---

转载请注明出处：[来自hualei的博客www.hualeihl.com](www.hualeihl.com)

最近用OC封装了一个日历控件，可以根据输入的月数，取出自当前月开始N个月的日期数据并展示，同时可选择日期，这里同时封装了一个日历工具类`HLCalendar`
>由于`NSCalendar`类很耗性能，所以我这里将它封装成了一个单例，避免重复创建

## HLCalendar
其中主要的两个实例方法名如下

``` Objective-C
/**
 获取自指定日期开始的月数据

 @param date 从该日期所在月开始获取
 @param monthes 获取的月的个数
 @return 月数据
 */
- (NSMutableArray *)monthsWithDate:(NSDate *)date monthes:(int)monthes;

/**
 获取指定日期的日历数据
 
 @param monthes 获取的月的个数
 @param fromDate 从该日期开始获取
 @return 日历数据
 */
- (NSMutableDictionary *)calendarDataWithMonthes:(int)monthes fromDate:(NSDate *)fromDate;
```

具体实现：

``` Objective-C
- (NSMutableArray *)monthsWithDate:(NSDate *)date monthes:(int)monthes {
    
    if (monthesC == monthes && monthesArr.count == monthes) {
        return monthesArr;
    }
    
    monthesC = monthes;
    
    NSMutableArray *months = [NSMutableArray array];
    
    NSCalendar * calendar = [NSCalendar calendarWithIdentifier:NSCalendarIdentifierGregorian];
    
    for (int i = 0; i < monthes; i++) {
        NSDateComponents *comps = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth fromDate:date];
        
        [comps setYear:0];
        [comps setMonth:i];
        NSDate *newdate = [calendar dateByAddingComponents:comps toDate:date options:0];
        
        NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
        formatter.dateFormat = @"yyyy年MM月";
        
        NSString *str = [NSString stringWithFormat:@"%@",[formatter stringFromDate:newdate]];
        
        [months addObject:str];
    }
    
    monthesArr = [months mutableCopy];
    
    return monthesArr;
}

- (NSMutableDictionary *)calendarDataWithMonthes:(int)monthes fromDate:(NSDate *)fromDate {
    
    if (monthesC == monthes && dateSource.allKeys.count == monthes) {
        return dateSource;
    }
    
    monthesC = monthes;
    
    NSMutableDictionary *calendarData = [NSMutableDictionary dictionary];
    
    for (int i = 0; i < monthes; i++) {
        
        NSDate *curMonthDate = [self dateWithDateStr:[self monthsWithDate:fromDate monthes:monthes][i] format:@"yyyy年MM月"];
        
        // 该日期对应的月的首日是周几
        NSInteger firstWeekday = [self weekdayOfFirstDayInDate:curMonthDate];
        // 该日期对应的月的日期数
        NSInteger totalDaysOfMonth = [self totalDaysInMonthOfDate:curMonthDate];
        
        // 取出字典对应月的数据数组
        NSMutableArray *monthArr = [NSMutableArray array];
        
        for (int j = 0; j < ceilf((totalDaysOfMonth + firstWeekday) / 7.0) * 7; j++) {
            
            if (j < firstWeekday) {
                [monthArr addObject:@{
                                      @"day":@"",
                                      @"chineseDay":@"",
                                      @"date":@""
                                      }];
            } else if (j >= totalDaysOfMonth + firstWeekday) {
                [monthArr addObject:@{
                                      @"day":@"",
                                      @"chineseDay":@"",
                                      @"date":@""
                                      }];
            } else {
                
                NSInteger day = j - firstWeekday + 1;
                
                NSDate *curDate = [self dateWithDateStr:[NSString stringWithFormat:@"%@%02ld日",[self monthsWithDate:fromDate monthes:monthes][i],(long)day] format:@"yyyy年MM月dd日"];
                
                NSDateComponents *comps0 = [[NSCalendar currentCalendar] components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay fromDate:[NSDate date]];
                NSDateComponents *comps1 = [[NSCalendar currentCalendar] components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay fromDate:curDate];
                
                if (comps0.year == comps1.year && comps0.month == comps1.month && comps0.day == comps1.day) {
                    [monthArr addObject:@{
                                          @"day":@"今天",
                                          @"chineseDay":[self chineseCalendarOfDate:curDate],
                                          @"date":curDate
                                          }];
                } else if (comps0.year == comps1.year && comps0.month == comps1.month && (comps0.day + 1) == comps1.day) {
                    [monthArr addObject:@{
                                          @"day":@"明天",
                                          @"chineseDay":[self chineseCalendarOfDate:curDate],
                                          @"date":curDate
                                          }];
                } else {
                    [monthArr addObject:@{
                                          @"day":[NSString stringWithFormat:@"%ld",(long)day],
                                          @"chineseDay":[self chineseCalendarOfDate:curDate],
                                          @"date":curDate
                                          }];
                }
            }
        }
        
        [calendarData setObject:monthArr forKey:[self monthsWithDate:fromDate monthes:monthes][i]];
    }
    
    dateSource = [calendarData mutableCopy];
    
    return dateSource;
}
```

UI部分就不赘述了，如下

![Paste_Image.gif](http://upload-images.jianshu.io/upload_images/1290529-4f60137d322c733c.gif?imageMogr2/auto-orient/strip)


源码：[gitHub地址](https://github.com/ThinkerH/HLCalendar)
