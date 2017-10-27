# rebrickable-database

![Build status badge](https://api.travis-ci.org/jncraton/rebrickable-database.png?branch=master)

This is set of scripts to create a local copy of the Rebrickable database. It uses the CSV downloads provided by [Rebrickable](https://rebrickable.com/downloads/) and is a relatively quick and easy way to create a local copy of all Lego parts and set inventories.

It includes custom views and indices to make working with this data more efficient.

## Getting started

Simply run `make`.

This will download the table data from Rebrickable and import it into an SQLite database.

## Example queries

### List all colors

```sql
select id, name 
from colors;
```

### Get all parts in a set

```sql
select part_num, color_id, quantity
from inventory_parts 
where inventory_id = (
  select id from inventories 
  where set_num = '10193-1'
  order by version desc
  limit 1
);
```

This may seem a little more complicated than it needs to be. Rebrickable maintains multiple versions of set inventories. This is a great feature, but we often want only the most recent version. More succinctly, you can use the `set_parts` view included in the project:

```sql
select part_num, color_id, quantity
from set_parts
where set_num = '10193-1';
```

### List top 10 parts based on the number of inventories they are in

Part numbers

```sql
select 
  part_num,
  sum(quantity) as total_quantity 
from inventory_parts
group by part_num
order by total_quantity desc
limit 10;
```

Including names

```sql
select 
  parts.part_num, 
  name, 
  sum(quantity) as total_quantity 
from inventory_parts
join parts on parts.part_num = inventory_parts.part_num
group by inventory_parts.part_num
order by total_quantity desc
limit 10;
```

### More examples

These examples, along with others, are available in the runable SQLite script `examples.sql`.
