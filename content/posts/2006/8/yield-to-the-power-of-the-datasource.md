+++
title = "Yield to the power of the DataSource"
date = 2006-08-02T15:13:00.000-05:00
updated = 2007-11-18T23:56:00.637-06:00
draft = false
url = '/2006/08/yield-to-power-of-datasource.html'
tags = ["DateTime",".Net","C#","DataSource","Asp.Net"]
+++

The new declarative model for data sources in ASP.Net 2.0 is very seductive in its simplicity. I've all-too-often seen people doing date-picker and time-pickers by listing the times individually in `ListItem`s. This leads to pages that have to be manually revisited if you change the range (start-time and end-time) or the increment (e.g. on-the-hour, on-the-half, etc.). It would be really cool to be able to drive these lists declaratively and bind them using the standard `DataSourceID` logic.

Using the new C# `yield` keyword it is trivial to return an `IEnumerable<DateTime>` that lists all the times (with a specified increment in minutes), days, weeks, months, etc. in a date range. I give you DateTimeDataSource:

```
using System;
using System.Collections.Generic;
using System.ComponentModel;
 
[DataObject(true)]
public class DateTimeDataSource
{
    private TimeSpan _DayStart = TimeSpan.FromHours(8); // default to 8am
    private TimeSpan _DayEnd = TimeSpan.FromHours(18); // default to 6pm
 
    [Browsable(true)]
    public TimeSpan DayStart
    {
        get { return _DayStart; }
        set { _DayStart = value; }
    }
 
    [Browsable(true)]
    public TimeSpan DayEnd
    {
        get { return _DayEnd; }
        set { _DayEnd = value; }
    }
 
    public static DateTime WeekStart(DateTime date)
    {
       return date.Date.AddDays(-(int)date.DayOfWeek);
    }
 
    public static DateTime WeekEnd(DateTime date)
    {
        return date.Date.AddDays(7).AddTicks(-1);
    }
 
    public static DateTime MonthStart(DateTime date)
    {
        return date.Date.AddDays(1 - date.Day);
    }
 
    public static DateTime MonthEnd(DateTime date)
    {
        return date.Date.AddMonths(1).AddTicks(-1);
    }
 
    public static DateTime YearStart(DateTime date)
    {
        return new DateTime(date.Year, 1, 1);
    }
 
    public static DateTime YearEnd(DateTime date)
    {
        return new DateTime(date.Year + 1, 1, 1).AddTicks(-1);
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, true)]
    public IEnumerable<DateTime> ThisWeek()
    {
        DateTime start = WeekStart(DateTime.Today);
        DateTime end = WeekEnd(start);
        return Any(start, end, TimeSpan.FromDays(1));
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, true)]
    public IEnumerable<DateTime> ThisMonth()
    {
        DateTime start = MonthStart(DateTime.Today);
        DateTime end = MonthEnd(start);
        return Any(start, end, TimeSpan.FromDays(1));
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, true)]
    public IEnumerable<DateTime> Today(int minuteIncrement)
    {
        return Any(DateTime.MinValue, DateTime.MaxValue, minuteIncrement);
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, false)]
    public IEnumerable<DateTime> Today(TimeSpan increment)
    {
        return Any(DateTime.MinValue, DateTime.MaxValue, increment);
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, true)]
    public IEnumerable<DateTime> Weeks(int numberOfWeeks)
    {
        DateTime start = WeekStart(DateTime.Today);
        DateTime end = start + TimeSpan.FromDays(7 * numberOfWeeks);
        return Weeks(start, end);
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, false)]
    public IEnumerable<DateTime> Weeks(DateTime start, int numberOfWeeks)
    {
        DateTime end = start + TimeSpan.FromDays(7 * numberOfWeeks);
        return Weeks(start, end);
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, false)]
    public IEnumerable<DateTime> Weeks(DateTime start, DateTime end)
    {
        return Any(start, end, TimeSpan.FromDays(7));
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, true)]
    public IEnumerable<DateTime> Any(DateTime start, DateTime end, int minuteIncrement)
    {
        return Any(start, end, TimeSpan.FromMinutes(minuteIncrement));
    }
 
    [DataObjectMethod(DataObjectMethodType.Select, false)]
    public IEnumerable<DateTime> Any(DateTime start, DateTime end, TimeSpan increment)
    {
        if (start == DateTime.MinValue || start.Ticks == 0)
        {
            start = DateTime.Today.Date + DayStart;
        }
 
        if (end == DateTime.MaxValue || end.Ticks == 0)
        {
            end = DateTime.Today.Date + DayEnd;
        }
 
        for (DateTime current = start; current <= end; current += increment)
        {
            yield return current;
        }
    }
}
```

Use this just like you would any other data source, for example, to build a `RadioButtonList` for the normal working hours (8am to 6pm, see notes below), you can do this:

```
<asp:ObjectDataSource ID="Hours" runat="server" CacheDuration="Infinite" EnableCaching="true"
        TypeName="DateTimeDataSource" SelectMethod="Today">
    <SelectParameters>
        <asp:Parameter Name="minuteIncrement" Type="Int32" Direction="Input" DefaultValue="30" />
    </SelectParameters>
</asp:ObjectDataSource>
<asp:RadioButtonList ID="RadioButtonTimeFrom" runat="server"
        DataSourceID="Hours" DataTextFormatString="{0:t}" />
```

### Notes:

*   The properties for `DayStart` and `DayEnd` let you change the default start and end time for time-pickers.
*   The XXXXStart and XXXXEnd static methods are there to simplify the generation of date ranges.
*   This isn't necessarily completely correct with other calendar forms.

**UPDATE**: Blogger ate my || in the code of Any

---

### Comments

#### That is cool man...I like that. Thanks…

[Unknown](https://www.blogger.com/profile/17304139413081012567 "noreply@blogger.com") - <time datetime="2007-06-07T04:29:00.000-05:00">Jun 4, 2007</time>

That is cool man...I like that. Thanks...
---
