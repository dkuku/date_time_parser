# DateTimeParser

[![Hex.pm Version](http://img.shields.io/hexpm/v/date_time_parser.svg)](https://hex.pm/packages/date_time_parser)
[![Hex docs](http://img.shields.io/badge/hex.pm-docs-blue.svg?style=flat)](https://hexdocs.pm/date_time_parser)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE.md)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v1.4%20adopted-ff69b4.svg)](./CODE_OF_CONDUCT.md)

DateTimeParser is a tokenizer for strings that attempts to parse into a
DateTime, NaiveDateTime if timezone is not determined, Date, or Time.

The biggest ambiguity between datetime formats is whether it's `ymd` (year month
day), `mdy` (month day year), or `dmy` (day month year); this is resolved by
checking if there are slashes or dashes. If slashes, then it will try `dmy`
first. All other cases will use the international format `ymd`. Sometimes, if
the conditions are right, it can even parse `dmy` with dashes if the month is a
vocal month (eg, `"Jan"`).

If the string consists of only numbers, then we will try two other parsers
depending on the number of digits: [Epoch] or [Serial]. Otherwise, we'll try the
tokenizer.

If the string is 10-11 digits with optional precision, then we'll try to parse
it as a Unix [Epoch] timestamp.

If the string is 1-5 digits with optional precision, then we'll try to parse it
as a [Serial] timestamp (spreadsheet time) treating 1899-12-31 as 1. This will
cause Excel-produced dates from 1900-01-01 until 1900-03-01 to be incorrect, as
they really are.

|digits|parser|range|notes|
|---|----|---|---|
|1-5|Serial|low = `1900-01-01`, high = `2173-10-15`. Negative numbers go to `1626-03-17`|Floats indicate time. Integers do not.|
|6-9|Tokenizer|any|This allows for "20190429" to be parsed as `2019-04-29`|
|10-11|Epoch|low = `1976-03-03T09:46:40`, high = `5138-11-16 09:46:39`|If padded with 0s, then it can capture entire range. Negative numbers not yet supported|

[Epoch]: https://en.wikipedia.org/wiki/Unix_time
[Serial]: https://support.office.com/en-us/article/date-systems-in-excel-e7fe7167-48a9-4b96-bb53-5612a800b487

## Planned Breaking Changes

* `parse_datetime` currently assumes `00:00:00` time if it cannot be determined.
    This will likely change in a future version because it's better to have no
    information than have wrong information. If you want to assume `00:00:00`,
    that's fine, but this library shouldn't assume it for you, or at least make
    it an option.
* `parse_datetime` currently defaults to converting to UTC when the timezone is
    known. This default may change to keep the original timezone information.
    This will help for future timestamps since timezone rules change; converting
    to UTC too early may use rules that become outdated by the time the
    timestamp arrives. The option to convert to UTC will remain, but may not be
    default.
* Introduce `parse` to parse as much as it can, but return any of the structs,
    `%DateTime{}` `%NaiveDateTime{}` `%Date{}` or `%Time{}`. It would be up to
    you to match on what the result is and do what you will. If you know you
    want the one specific struct, then you can continue to use the more-specific
    functions like `parse_date`.

## Required reading

* [Elixir DateTime docs](https://hexdocs.pm/elixir/DateTime.html)
* [Elixir NaiveDateTime docs](https://hexdocs.pm/elixir/NaiveDateTime.html)
* [Elixir Date docs](https://hexdocs.pm/elixir/Date.html)
* [Elixir Time docs](https://hexdocs.pm/elixir/Time.html)
* [Elixir Calendar docs](https://hexdocs.pm/elixir/Calendar.html)
* [How to save datetimes for future events (when UTC is not the right answer)](http://www.creativedeletion.com/2015/03/19/persisting_future_datetimes.html)
  * tldr: rules change, so don't convert to UTC too early. The future might
      change the timezone conversion rules.

## Documentation

[Online Documentation](https://hexdocs.pm/date_time_parser)

## Examples

```elixir
iex> DateTimeParser.parse_datetime("19 September 2018 08:15:22 AM")
{:ok, ~N[2018-09-19 08:15:22]}

iex> DateTimeParser.parse_datetime("2034-01-13")
{:ok, ~N[2034-01-13 00:00:00]}

iex> DateTimeParser.parse_date("2034-01-13")
{:ok, ~D[2034-01-13]}

iex> DateTimeParser.parse_date("01/01/2017")
{:ok, ~D[2017-01-01]}

iex> DateTimeParser.parse_datetime("1/1/18 3:24 PM")
{:ok, ~N[2018-01-01T15:24:00]}

iex> DateTimeParser.parse_datetime("1/1/18 3:24 PM", assume_utc: true)
{:ok, ~U[2018-01-01T15:24:00Z]}
# the ~U is a DateTime sigil introduced in Elixir 1.9.0

iex> DateTimeParser.parse_datetime(~s|"Dec 1, 2018 7:39:53 AM PST"|)
{:ok, ~U[2018-12-01T14:39:53Z]}
# Notice that the date is converted to UTC by default

iex> {:ok, datetime} = DateTimeParser.parse_datetime(~s|"Dec 1, 2018 7:39:53 AM PST"|, to_utc: false)
iex> datetime
#DateTime<2018-12-01 07:39:53-07:00 PDT PST8PDT>

iex> DateTimeParser.parse_time("10:13pm")
{:ok, ~T[22:13:00]}

iex> DateTimeParser.parse_time("10:13:34")
{:ok, ~T[10:13:34]}

iex> DateTimeParser.parse_datetime(nil)
{:error, "Could not parse nil"}
```

[See more examples automatically generated by the tests](./EXAMPLES.md)

## Installation

Add `date_time_parser` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:date_time_parser, "~> 0.1.4"}
  ]
end
```

## Changelog

[View Changelog](./CHANGELOG.md)

## Contributing

[How to contribute](./CONTRIBUTING.md)

## Special Thanks

[<img src="https://www.taxjar.com/img/lander/logo.svg" height=75 />](https://www.taxjar.com)
