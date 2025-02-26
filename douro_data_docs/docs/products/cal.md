![Tzolk'in](../assets/MAYA-g-log-cal-D15-Men-cdxW.png)

# Cal: The Calendar Dimension For You

In less than 15 minutes, Cal can be configured and drastically improving your analytic ability. Let us help you unlock powerful insights within the data you already have.

Additionally, Cal is idempotent, meaning you can generate your calendar as many times as you want without it changing the results.

Let's get started!

## Types of Calendars

We currently support the following calendar types:

- **Basic:** A generic date dimension
- **Fiscal:** A fiscal date dimension
<!-- - ISO (A calendar dimension that follows the ISO 8601 standard) -->
- **Nonprofit:** A fiscal date dimension that follows the standard for nonprofits
- **U.S.A. Government:** A fiscal date dimension that follows the standard of the US Government
- **445/454/544:** Heavily used in retail, a fiscal calendar designed around comparable reporting periods. Also handles the 53rd week.

Although there are slight differences in the setup of each, the general process is outlined below:

1. Define the start and end of the calendar
2. Select if and where holidays should be included from
3. Select the account timezone
4. Choose the first month of quarter 1 (_fiscal only_)
5. Choose the day your week starts (_fiscal only_)

> **_NOTE:_** The 445 has some extra unique characteristics, so is discussed in further detail in the following section.

## The 445 Calendar

The 445 calendar (along with the siblings 454 and 544) is a fiscal calendar that is heavily used in retail. It is designed to allow for meaningful and consistent reporting periods that can be compared and aggregrated accurately as well as to manage the potential 53rd week of the year, which is often a week of extra sales.

In practice, this means that the fiscal calendar will often start earlier than the standard Gregorian calendar.

Using an example, let's say the fiscal year starts on January 1st and the company starts the week on a Saturday. Depending on the year, the fiscal year could start back up to 6 days before the standard calendar year. So for the year 2015, the fiscal year would actually start on Saturday, December 27th!

To minimize some of the potential confusion around the misalignment of a fiscal and gregorian calendar, we push the actual start date of your calendar dimension to the first year where both calendars align. In our example case, that would mean going back to 2011.

## Using Cal

Once installed, you can start using Cal straight away. Let's start with a simpler use case:

```sql
select
your_table.*,
cal_dim.*

from your_table

inner join douro_data.douro_cal_dimension as cal_dim
on your_table.date_field = cal_dim.sql_date
```

In a more complex example, we can use the dimension to check a previous fiscal year quarter:

```sql
select
your_data.*

from your_data

inner join douro_data.douro_cal_dimension
on your_data.date_field = douro_data.douro_cal_dimension.sql_date
and douro_data.douro_cal_dimension.fiscal_ly_qtd = true
```

## Daily Updates

Cal is designed to be updated daily and ships with the ability to do so without any intervention. This means that you can expect your calendar dimension to be fully up to date at the start of each day.

## Usage Scenarios

The calendar dimension table can be used in various ways to support time-based analyses and reporting:

1. **Filtering and Grouping**: Use the date-related fields (e.g., `DATE_KEY`, `SQL_DATE`, `CAL_YEAR_NUM`, `CAL_QUARTER_NUM`, `CAL_MONTH_NUM`) to filter and group data based on time periods.
1. **Trend Analysis**: Analyze trends over time using the date-related fields and the associated metrics (e.g., sales, revenue, costs).
1. **Seasonality Analysis**: Identify seasonal patterns by examining the data at different granularities (e.g., day, week, month, quarter, year).
1. **Holiday and Weekend Analysis**: Analyze the impact of holidays and weekends on various business metrics using the `WEEKEND` and `HOLIDAY` flags.
1. **Fiscal Calendar Support**: The table includes fiscal calendar-related fields (e.g., `FISCAL_YEAR_NUMBER`, `FISCAL_QUARTER_NUMBER`, `FISCAL_MONTH_NUMBER`) to support reporting and analysis based on a fiscal calendar.

## Column definitions

| Column Identifier      | Name                                    | Definition                                                                                              |
| ---------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| DATE_KEY               | Date Key                                | A unique identifier for each date, typically an integer value representing the date in YYYYMMDD format. |
| SQL_DATE               | SQL Date                                | The date in the standard SQL date format (YYYY-MM-DD).                                                  |
| DATE_NAME              | Date Name                               | The date in the MM/DD/YYYY format.                                                                      |
| SORT_DATE              | Sort Date                               | The date in the YYYY/MM/DD format, designed for sorting and ordering purposes.                          |
| CAL_DAY_NUM            | Cal Day Number                          | The sequential day number, starting from 1 on January 1st of the earliest year in the data.             |
| CAL_MONTH_NUM          | Cal Month Number                        | The sequential month number, starting from 1 for January.                                               |
| CAL_QUARTER_NUM        | Cal Quarter Number                      | The sequential quarter number, starting from 1 for the first quarter.                                   |
| CAL_YEAR_NUM           | Cal Year Number                         | The year number.                                                                                        |
| DAY_OF_WEEK_NUMBER     | Day of Week Number                      | The day of the week number, where 1 represents Sunday and 7 represents Saturday.                        |
| DAY_OF_WEEK_NAME       | Day of Week Name                        | The name of the day of the week.                                                                        |
| DAY_OF_MONTH_NUMBER    | Day of Month Number                     | The day of the month number, ranging from 1 to 31.                                                      |
| DAY_OF_MONTH_NAME      | Day of Month Name                       | The name of the day of the month.                                                                       |
| DAY_OF_YEAR_NUMBER     | Day of Year Number                      | The sequential day of the year number, starting from 1 on January 1st.                                  |
| DAY_OF_YEAR_NAME       | Day of Year Name                        | The name of the day of the year.                                                                        |
| DAY_NAME               | Day Name                                | The full name of the day of the week.                                                                   |
| DAY_SHORT_NAME         | Day Short Name                          | The abbreviated name of the day of the week.                                                            |
| WEEKEND                | Weekend                                 | A boolean flag indicating whether the day is a weekend (true) or a weekday (false).                     |
| HOLIDAY                | Holiday                                 | A boolean flag indicating whether the day is a holiday (true) or not (false).                           |
| HOLIDAY_NAME           | Holiday Name                            | The name of the holiday, if the day is a holiday.                                                       |
| WEEK_OF_MONTH_NUMBER   | Week of Month Number                    | The week of the month number, ranging from 1 to 5.                                                      |
| WEEK_OF_MONTH_NAME     | Week of Month Name                      | The name of the week of the month.                                                                      |
| WEEK_OF_YEAR_NUMBER    | Week of Year Number                     | The week of the year number, ranging from 1 to 52 or 53.                                                |
| WEEK_OF_YEAR_NAME      | Week of Year Name                       | The name of the week of the year.                                                                       |
| WEEK_START_DATE        | Week Start Date                         | The start date of the week.                                                                             |
| WEEK_END_DATE          | Week End Date                           | The end date of the week.                                                                               |
| MONTH_OF_YEAR_NUMBER   | Month of Year Number                    | The month of the year number, ranging from 1 to 12.                                                     |
| MONTH_OF_YEAR_NAME     | Month of Year Name                      | The name of the month of the year.                                                                      |
| MONTH_NAME             | Month Name                              | The full name of the month.                                                                             |
| MONTH_SHORT_NAME       | Month Short Name                        | The abbreviated name of the month.                                                                      |
| MONTH_START_DATE       | Month Start Date                        | The start date of the month.                                                                            |
| MONTH_END_DATE         | Month End Date                          | The end date of the month.                                                                              |
| QUARTER_OF_YEAR_NUMBER | Quarter of Year Number                  | The quarter of the year number, ranging from 1 to 4.                                                    |
| QUARTER_OF_YEAR_NAME   | Quarter of Year Name                    | The name of the quarter of the year.                                                                    |
| QUARTER_START_DATE     | Quarter Start Date                      | The start date of the quarter.                                                                          |
| QUARTER_END_DATE       | Quarter End Date                        | The end date of the quarter.                                                                            |
| YEAR_NUMBER            | Year Number                             | The year number.                                                                                        |
| YEAR_NAME              | Year Name                               | The name of the year.                                                                                   |
| YEAR_START_DATE        | Year Start Date                         | The start date of the year.                                                                             |
| YEAR_END_DATE          | Year End Date                           | The end date of the year.                                                                               |
| `WTD`                  | Week to Date                            | Calendar dates for the week up to the current date                                                      |
| `WTY`                  | Week to Yesterday                       | Calendar dates for the week up to the previous day                                                      |
| `MTD`                  | Month to Date                           | Calendar dates for the month up to the current date                                                     |
| `MTY`                  | Month to Yesterday                      | Calendar dates for the month up to yesterday                                                            |
| `QTD`                  | Quarter to Date                         | Calendar dates for the quarter up to the current date                                                   |
| `QTY`                  | Quarter to Yesterday                    | Calendar dates for the quarter up to yesterday                                                          |
| `YTD`                  | Year to Date                            | Calendar dates for the year up to the current date                                                      |
| `YTY`                  | Year to Yesterday                       | Calendar dates for the year up to yesterday                                                             |
| `LY_WTD`               | Last Year - Week to Date                | Calendar dates for the previous year week up to the previous year date                                  |
| `LY_WTY`               | Last Year - Week to Yesterday           | Calendar dates for the previous year week up to the previous year day                                   |
| `LY_MTD`               | Last Year - Month to Date               | Calendar dates for the previous year month up to the previous year date                                 |
| `LY_MTY`               | Last Year - Month to Yesterday          | Calendar dates for the previous year month up to the previous year day                                  |
| `LY_QTD`               | Last Year - Quarter to Date             | Calendar dates for the previous year quarter up to the previous year date                               |
| `LY_QTY`               | Last Year - Quarter to Yesterday        | Calendar dates for the previous year quarter up to the previous year day                                |
| `LY_YTD`               | Last Year - Year to Date                | Calendar dates for the previous year up to the previous year date                                       |
| `LY_YTY`               | Last Year - Year to Yesterday           | Calendar dates for the previous year up to the previous year day                                        |
| `THIS_YEAR_M01`        | This Year Month 1                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M02`        | This Year Month 2                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M03`        | This Year Month 3                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M04`        | This Year Month 4                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M05`        | This Year Month 5                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M06`        | This Year Month 6                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M07`        | This Year Month 7                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M08`        | This Year Month 8                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M09`        | This Year Month 9                       | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M10`        | This Year Month 10                      | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M11`        | This Year Month 11                      | A boolean field representing if the current month is active                                             |
| `THIS_YEAR_M12`        | This Year Month 12                      | A boolean field representing if the current month is active                                             |
| `LAST_YEAR_M01`        | Last Year Month 1                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M02`        | Last Year Month 2                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M03`        | Last Year Month 3                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M04`        | Last Year Month 4                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M05`        | Last Year Month 5                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M06`        | Last Year Month 6                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M07`        | Last Year Month 7                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M08`        | Last Year Month 8                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M09`        | Last Year Month 9                       | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M10`        | Last Year Month 10                      | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M11`        | Last Year Month 11                      | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `LAST_YEAR_M12`        | Last Year Month 12                      | A boolean field identifying if the field is the same month of the previous year of the current month    |
| `FISCAL_WTD`           | Fiscal Week to Date                     | Fiscal Calendar dates for the week up to the current date                                               |
| `FISCAL_WTY`           | Fiscal Week to Yesterday                | Fiscal Calendar dates for the week up to the previous day                                               |
| `FISCAL_PTD`           | Fiscal Period to Date                   | Fiscal Calendar dates for the period up to the current date                                             |
| `FISCAL_PTY`           | Fiscal Period to Yesterday              | Fiscal Calendar dates for the period up to yesterday                                                    |
| `FISCAL_QTD`           | Fiscal Quarter to Date                  | Fiscal Calendar dates for the quarter up to the current date                                            |
| `FISCAL_QTY`           | Fiscal Quarter to Yesterday             | Fiscal Calendar dates for the quarter up to yesterday                                                   |
| `FISCAL_YTD`           | Fiscal Year to Date                     | Fiscal Calendar dates for the year up to the current date                                               |
| `FISCAL_YTY`           | Fiscal Year to Yesterday                | Fiscal Calendar dates for the year up to yesterday                                                      |
| `FISCAL_LY_WTD`        | Fiscal Last Year - Week to Date         | Fiscal Calendar dates for the previous year week up to the previous year date                           |
| `FISCAL_LY_WTY`        | Fiscal Last Year - Week to Yesterday    | Fiscal Calendar dates for the previous year week up to the previous year day                            |
| `FISCAL_LY_PTD`        | Fiscal Last Year - Period to Date       | Fiscal Calendar dates for the previous year period up to the previous year date                         |
| `FISCAL_LY_PTY`        | Fiscal Last Year - Period to Yesterday  | Fiscal Calendar dates for the previous year period up to the previous year day                          |
| `FISCAL_LY_QTD`        | Fiscal Last Year - Quarter to Date      | Fiscal Calendar dates for the previous year quarter up to the previous year date                        |
| `FISCAL_LY_QTY`        | Fiscal Last Year - Quarter to Yesterday | Fiscal Calendar dates for the previous year quarter up to the previous year day                         |
| `FISCAL_LY_YTD`        | Fiscal Last Year - Year to Date         | Fiscal Calendar dates for the previous year up to the previous year date                                |
| `FISCAL_LY_YTY`        | Fiscal Last Year - Year to Yesterday    | Fiscal Calendar dates for the previous year up to the previous year day                                 |
| `THIS_YEAR_P01`        | This Year Period 1                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P02`        | This Year Period 2                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P03`        | This Year Period 3                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P04`        | This Year Period 4                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P05`        | This Year Period 5                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P06`        | This Year Period 6                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P07`        | This Year Period 7                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P08`        | This Year Period 8                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P09`        | This Year Period 9                      | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P10`        | This Year Period 10                     | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P11`        | This Year Period 11                     | A boolean field representing if the current period is active                                            |
| `THIS_YEAR_P12`        | This Year Period 12                     | A boolean field representing if the current period is active                                            |
| `LAST_YEAR_P01`        | Last Year Period 1                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P02`        | Last Year Period 2                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P03`        | Last Year Period 3                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P04`        | Last Year Period 4                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P05`        | Last Year Period 5                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P06`        | Last Year Period 6                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P07`        | Last Year Period 7                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P08`        | Last Year Period 8                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P09`        | Last Year Period 9                      | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P10`        | Last Year Period 10                     | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P11`        | Last Year Period 11                     | A boolean field identifying if the field is the same period of the previous year of the current period  |
| `LAST_YEAR_P12`        | Last Year Period 12                     | A boolean field identifying if the field is the same period of the previous year of the current period  |

## Further Info

By using the calendar dimension table, you can easily perform sophisticated time-based analyses and create consistent, reliable reports across your data warehouse.

Please reach out to us with any questions or suggestions at [support@dourodata.com](mailto:support@dourodata.com) or use the [support form](https://www.dourodata.com/support)!
