
## Ejercicios de consultas a tablas

```sql
select count (title) from film;

select count (*) from film;

select avg(rental_rate) from film;

select title, 
	rental_rate, 
	replacement_cost, 
	round(replacement_cost/rental_rate, 2) as percentage 
from public.film;

select title,
	ceil(replacement_cost/rental_rate) as rents_to_profit
from public.film;

select *
from film
where 2 < rental_rate and rental_rate > 3;

select title, rental_rate
from public.film
where rental_rate between 2 and 3;

select title, 
	rental_rate, 
	release_year
from public.film
where title like 'A%s'
	and rental_rate between 2 and 3
	and release_year = 2006
	and title is not null;

select first_name,
	last_name
from public.actor
where first_name like 'A%';

select title,
	rental_rate
from public.film
where rental_rate > 10;

select title,
	rental_rate
from public.film
where rental_rate between 5 and 10;

select title,
	rental_rate,
	f.length
from public.film as f
where rental_rate < 5
	and length < 100;
	
select title, 
	rental_rate
from public.film
where title like 'Gi%';

select title, 
	rating,
	f.length
from public.film as f
where title = 'Ali Forever';

select title,
	rental_rate
from public.film
where rental_rate is null;

select title,
	f.length
from public.film as f
order by length
limit 5;

select rating,
	count(distinct title) as películas,
	round(avg(rental_rate),2) as precio_medio,
	min(rental_rate) as precio_más_bajo,
	max(rental_rate) as precio_más_alto,
	round(avg(f.length),2) as duración,
	min(release_year) as más_antigua,
	max(release_year) as más_nueva
from public.film as f
group by rating
having avg(f.length) > 110;

select rating,
	count(distinct title) as películas,
	round(avg(rental_rate),2) as precio_medio,
	round(avg(f.length),2) as duración
from public.film as f
group by rating
having count(distinct title) > 200
	and avg(rental_rate) > 3
	and avg(f.length) > 115;
	
select customer.first_name,
	customer.last_name,
	address.address
from public.customer customer
	inner join public.address address 
	on customer.address_id = address.address_id
limit 10;

-- ficha de cliente: nombre, país, ciudad y dirección
select customer.first_name,
	customer.last_name,
	country.country,
	city.city,
	address.address
from public.customer customer
	left join public.address address 
		on customer.address_id = address.address_id
	left join public.city city
		on address.city_id = city.city_id
	left join public.country country
		on city.country_id = country.country_id
limit 10;

-- ¿cuántos actores tiene cada película?
select f.title,
	count(distinct a.actor_id) as actores
from public.actor a
	left join public.film_actor fa
		on a.actor_id = fa.actor_id
	left join public.film f
		on fa.film_id  = f.film_id
group by f.title
order by count(distinct a.actor_id) desc;

-- ¿cuáles son las películas que tienen más de dos actores?
select f.title,
	count(distinct a.actor_id) as actores
from public.actor a
	left join public.film_actor fa
		on a.actor_id = fa.actor_id
	left join public.film f
		on fa.film_id  = f.film_id
group by f.title
having count(distinct a.actor_id) > 2
order by count(distinct a.actor_id) desc;

-- ¿cuál es la película que tiene más actores?
select f.title,
	count(distinct a.actor_id) as actores
from public.actor a
	left join public.film_actor fa
		on a.actor_id = fa.actor_id
	left join public.film f
		on fa.film_id  = f.film_id
group by f.title
order by count(distinct a.actor_id) desc
limit 1;
```

## Ejercicios de creación de tablas y de inserción de datos

```sql
-- crear tabla
create table if not exists public.reviews(
	film_id integer not null,
	customer_id integer not null,
	review_date date not null,
	review_description character varying(50) not null,
	constraint film_customer_pkey
		primary key (film_id, customer_id) -- una sola primary key compuesta por dos campos
	);
```

```sql
-- insertar datos en nuestra tabla reviews
insert into public.reviews (film_id, customer_id, review_date, review_description)
	values (1234,5678,'01-01-2024','ok');

-- sobreescribir un campo
update public.reviews
set review_description = 'good'
where film_id = 1234

-- borrar un registro
delete
from public.reviews
where film_id = 1234

-- borrar tabla entera
drop table public.reviews

--función ALTER 

Añade una nueva columna a la tabla de reviews que te parezca interesante. Por ejemplo, el número de
estrellas que le darías a una película. Puedes llamarle “review_stars” y con datatype int2

ALTER TABLE reviews_ng 
ADD COLUMN review_stars INT2


Renombra una de las columnas de la tabla de reviews. Por ejemplo, renombra la columna
“review_description” a “review_opinion”

ALTER TABLE reviews_ng 
RENAME COLUMN review_description
to review_opinion ;


Cambia el tipo de dato de la columna de “review_stars” a varchar. Ahora es
in int2.

ALTER TABLE reviews_ng 
ALTER COLUMN review_stars TYPE VARCHAR;


Borra la columna de “review_stars”

Eliminar 
ALTER TABLE reviews_ng 
DROP COLUMN review_stars;

Views: para los que toman decisiones

select * from actor_info ai

Crear una nueva vista 
CREATE VIEW my_view_of_actor AS
SELECT actor_id, first_name, last_name, last_update
FROM public.actor
where first_name is not null;

Crear mi propia view 
CREATE VIEW my_view_martamr AS
SELECT actor_id, first_name, last_name, last_update
FROM public.actor
where first_name is not null;

select * from my_view_martamr

ejemplo condicional 

select length,
case when length > 100 then 'PELICULA_LARGA'
when length >50 and Length < 99 then 'PELICULA_MEDIA'
else 'PELICULA_CORTA' END
from film
 

Concatenar
select first_name, last_name, first_name || ' ' || last_name as nombre_completo, concat(first_name, ' ', last_name)
from actor a

Para sacar los clientes que más consumen: 

select concat(first_name, ' ', last_name) as nombre_completo, sum(p.amount)
from payment p
inner join customer c on c.customer_id = p.customer_id
group by 1
order by 2 desc;
 
Fechas

SELECT (return_date::date - rental_date::date) AS difference_in_days,
return_date::date, rental_date::date, rental_id::varchar
from rental r ; 

Query top 5 clientes 

CREATE or replace view top5 AS
select concat(first_name, ' ', last_name) as nombre_completo, sum(p.amount)
from payment p
inner join customer c on c.customer_id = p.customer_id
group by 1
order by 2 desc
limit 5;
 
CREATE or replace view down5 AS
select concat(first_name, ' ', last_name) as nombre_completo, sum(p.amount)
from payment p
inner join customer c on c.customer_id = p.customer_id
group by 1
order by 2 asc
limit 5;

Para unir 2 views:

select * from down5
union all 
select * from top5 


Hacer KPI de la vista de top 5: 

create view kpis as
select avg(down5.sum) , 'down5' as tipo from down5
union all
select avg(top5.sum), 'top5' as tipo from top5 ;
 
select * from kpis ;
  

Subconsultascon WHERE: 

SELECT actor_id, first_name, last_name, last_update
FROM public.actor
where first_name in (select first_name from
public.actor where last_name ilike 'C%')

Ejemplo join vs subconsulta 

select sum(p.amount) from payment p
left join customer c on c.customer_id =p.customer_id
where c.first_name  ilike 'C%'
 
select sum(p.amount) from payment p
where customer_id in (
	select customer_id from customer c where first_name  ilike 'C%'
)


Subconsultas con with 

with my_sub_query as
(SELECT title, rental_rate, replacement_cost, round(replacement_cost / rental_rate, 2) as new_column
FROM public.film),
new_1 as
(SELECT title, rental_rate, replacement_cost, round(replacement_cost / rental_rate, 2) as new_column
FROM public.film)
select * from new_1 where title = 'Ali Forever'
 

Obtén haciendo una subconsulta en la cláusula WITH
- La suma de los amount que de los clientes que han pagado más
de 190€

Sub: 

with my_sub_query as 
(select customer_id, sum(amount) total
from paymnets 
group by 1

consulta: 
select * from sub 
where total> 190


como quedaría:

with sub as
(select customer_id, sum(amount) as total
from payment p 
group by customer_id
)
select * from sub where total >190; 


empleado del mes: 

with top_employer as
	(
	select p.staff_id as staff_id,
		sum(p.amount) as amount,
		count(distinct customer_id) as clientes
	from payment p
	group by p.staff_id
	order by sum(p.amount) desc
	limit 1
	),
	sub_address as (
	select a.address as address,
		s.staff_id as staff_id,
		(s.first_name || ' ' || s.last_name) as full_name
	from staff s
		left join address a
		on a.address_id = s.address_id
	),
	rental_duration as (
	select r.staff_id as staff_id,
		avg(r.return_date::date - r.rental_date::date) as rental_duration
	from rental r
	group by r.staff_id
	)
select s.full_name,
	s.address,
	t.clientes,
	r.rental_duration
from top_employer t
left join sub_address s
	on t.staff_id = s.staff_id
left join rental_duration r
	on r.staff_id = s.staff_id


