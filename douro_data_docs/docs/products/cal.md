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
- **445:** Heavily used in retail, a fiscal calendar designed around managing the 53rd week

Although there are slight differences in the setup of each, the general process is outlined below:

1. Define the start and end of the calendar
2. Select if and where holidays should be included from
3. Select the account timezone
4. Choose the first month of quarter 1 (_fiscal only_)
5. Choose the day your week starts (_fiscal only_)

> **_NOTE:_** The 445 has some extra unique characteristics, so is discussed in further detail in the following section.

## The 445 Calendar

The 445 calendar is a fiscal calendar that is heavily used in retail. It is designed to manage the 53rd week of the year, which is often a week of extra sales.

In practice, this means that the fiscal calendar will potentially start earlier than the standard calendar.

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

Cal is designed to be updated daily. This means that you can expect your calendar dimension to be fully up to date at the start of each day.
