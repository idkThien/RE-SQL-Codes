# RE-SQL-Codes

T-SQL queries for exploring and reporting on a Blackbaud Raiser's Edge 7 database in SQL Server. The main query combines constituent, spouse, address, solicit code, note, and constituent code data into one flattened result set for reporting and data cleanup.

## Repository contents

| File | Description |
|---|---|
| `constituent_profile.sql` | Main query: flattened constituent profile |
| `constit_address_explore.sql` | Quick look at the `CONSTIT_ADDRESS` table |
| `address_explore.sql` | Quick look at the `ADDRESS` table |
| `database-diagram.png` | SSMS diagram of the core tables |

## Database structure

![Raiser's Edge table relationships: RECORDS, CONSTIT_ADDRESS, ADDRESS](database-diagram.png)

| Table | Purpose | Key columns |
|---|---|---|
| `RECORDS` | One row per constituent record; also links spouses through a self-reference | `ID`, `CONSTITUENT_ID`, `FIRST_NAME`, `LAST_NAME`, `BIRTH_DATE`, `SPOUSE_ID` |
| `CONSTIT_ADDRESS` | Link table connecting constituents to addresses, with flags and date ranges | `CONSTIT_ID`, `ADDRESS_ID`, `PREFERRED`, `SENDMAIL`, `SEASONAL`, `DATE_FROM`, `DATE_TO` |
| `ADDRESS` | The address details themselves | `ID`, `ADDRESS_BLOCK`, `CITY`, `STATE`, `POST_CODE`, `COUNTY`, `COUNTRY` |

**How the tables connect**

- `RECORDS.ID` → `CONSTIT_ADDRESS.CONSTIT_ID`: one constituent can have many address links
- `CONSTIT_ADDRESS.ADDRESS_ID` → `ADDRESS.ID`: each link points to one address
- `RECORDS.SPOUSE_ID` → `RECORDS.ID`: self-join used to pull spouse details

## Main query: constituent profile

Starts from `RECORDS` and joins:

| Table | Used for |
|---|---|
| `RECORDS` (self-join) | Spouse details |
| `CONSTIT_ADDRESS` + `ADDRESS` | Address, preferred and send-mail flags, last changed date, info source |
| `CONSTITUENT_SOLICITCODES` | Solicit codes |
| `ConstituentNotepad` | Notes |
| `CONSTITUENT_CODES` | Constituent codes |

Raw numeric flags (inactive, deceased, anonymous, sex, title, etc.) are translated into readable values with `CASE` statements, and birth dates are formatted as `MM/DD/YYYY`.

## Exploring the tables

To inspect a table before writing joins, I started with a simple preview (SSMS generates these through *Select Top 1000 Rows*):

```sql
SELECT TOP (1000) *
FROM [YOUR_RE_DATABASE].[dbo].[CONSTIT_ADDRESS];

SELECT TOP (1000) *
FROM [YOUR_RE_DATABASE].[dbo].[ADDRESS]
ORDER BY ID;
```

## How to use

1. Open the files in SQL Server Management Studio (SSMS).
2. Replace `[YOUR_RE_DATABASE]` with the name of your Raiser's Edge database.
3. For the main query, add a filter in the `WHERE` clause (for example, a single constituent ID) to limit the results, then run it.

## Status and known limitations

This is a work in progress.

- **Some decoded values are assumptions.** The mappings for fields like `INACTIVE`, `SEX`, and `TITLE_1` are educated guesses, marked "Actual values UNKNOWN" in the code comments. Verify them against your own database before relying on them.
- **Rows can repeat.** A constituent with multiple addresses, notes, or codes returns multiple rows because of the one-to-many `LEFT JOIN`s.
- **Historical addresses may appear.** `CONSTIT_ADDRESS` stores date ranges and flags, so a future version should filter to the current, preferred address (using `PREFERRED`, `DATE_FROM`, and `DATE_TO`).
- **Not yet built:** gift summary, last gift, and attributes sections (placeholders in the code), decoding of constituent codes, and an age calculation (written but commented out).

## Privacy note

This repo contains only queries and a schema diagram, with no data. The main query returns personal information (names, birth dates, addresses, notes), so only run it against data you're authorized to access, and don't commit exports or results.

## Tools

T-SQL, SQL Server Management Studio (SSMS), Blackbaud Raiser's Edge 7
