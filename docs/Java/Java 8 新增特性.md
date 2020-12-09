# Java 8 新增特性

### 日期处理
| 类型 | 方法 | 结果 |
| --- | --- | --- |
| 当前日期 | `LocalDate.now()` | 2020-05-28 |
| 获取月份 | `localDate.getMonthValue()` | 5 |
| 获取年份 | `localDate.getYear()` | 2020 |
| 获取日期 | `localDate.getDayOfMonth()` | 28 |
| 获取星期 | `localDate.getDayOfWeek()` | THURSDAY |
| 创建日期 | `LocalDate.of(2020,5,28)` | 2020-05-28 |
| 生日检查 | `MonthDay.form(localDate)` | 05-28 |
| 获取当前时间 | `LocalTime.now()` | 23:29:12.146 |
| 增加小时 | `localTime.plusHours()` |  |
| 增加天数 | `localDate.plus(1, ChronoUnit.DAYS)` |  |
| 检查闰年 | `localDate.isLeapYear()` |  |
| 日期比较 | `Period.between()` |  |
| 时间戳 | `Instant.now()` | 2020-05-28T15:29:12.154Z |



### Lambda
