## Country Dimension

**Your comprehensive geographic reference**

Enhance your analytical capabilities with the country dimension table that leverages ISO 3166 standards. This dimension provides a robust, standardized framework for geographic data analysis, enabling precise reporting, seamless data integration, and simplified compliance across global business operations. Key benefits include consistent country representation, improved data quality, and the ability to generate granular insights by efficiently segmenting and analyzing data at the country level.

- Ideal for organizations seeking to:
- Standardize geographic data across systems
- Support detailed regional analytics
- Ensure global data compatibility
- Facilitate targeted strategic decision-making

Includes comprehensive ISO country code mappings, allowing for accurate and scalable international data management.

### Column Definitions

| Column Identifier | Definition                                                                                     | Sample Value             |
| ----------------- | ---------------------------------------------------------------------------------------------- | ------------------------ |
| name              | The generally used name for a country                                                          | United States            |
| alpha_2           | The two letter code used to identify a country                                                 | US                       |
| alpha_3           | The three letter code used to identify a country                                               | USA                      |
| common_name       | If applicable, a separate name used for the country. Filled with the name if not applicable.   | United States            |
| official_name     | If applicable, the official name used for the country. Filled with the name if not applicable. | United States of America |
| numeric_code      | Similar to alpha_3, a three digit code used to identify countries                              | 840                      |
| flag              | the UTF-8 (Unicode) emoji flag of the identified country.                                      | 🇺🇸                       |

### Usage Examples

**Example to pull from the dimension table**

```sql
select
country.name,
count(transaction.id) as transaction_counts
from your_transaction_table as transaction
inner join douro_data.country_dimension as country
on transaction.country_id = country.country_id
group by
country.name
```

**Example to insert into fact with fallback Kimball row**

```sql
select
source.id,
source.transaction_amount,
source.transaction_created,
coalesce(country.country_id, -1) as country_id
...
from source_table as source
left outer join douro_data.country_dimension as country
on source.alpha_2_country_code = country.alpha_2
```

### Data update policy

This is an important note copied over from the pycountry repository, which feeds the data for this dimension.

> **_NOTE:_** No changes to the data will be accepted into pycountry. This is a pure wrapper around the ISO standard using the `pkg-isocodes` database from Debian as is.
> If you need changes to the political situation in the world, please talk to the ISO or Debian people, not me.

### Further Info

By using the country dimension table, you can easily perform more sophisticated region-based analyses and create consistent, reliable reports across your data warehouse.

Please reach out to us with any questions or suggestions at [support@dourodata.com](mailto:support@dourodata.com) or use the [support form](https://www.dourodata.com/support)!
