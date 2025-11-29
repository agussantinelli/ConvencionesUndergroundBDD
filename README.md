<h1>📘 Convenciones Underground — Parcial y Recuperatorio de AD SQL</h1>
<p><strong>Resoluciones SQL – Parcial + Recuperatorio</strong></p>

<hr>

<h2>📎 Links Útiles</h2>
<ul>
    <li><a href="https://docs.google.com/document/d/11ITS_bJAtdjuZHidNuQigZq5awerU3Wy/edit">Enunciado MD</a></li>
    <li><a href="https://app.diagrams.net/#G1wqzrNx-f2fEjGteM0aAX06rToOOTKr7U">Resolución MD</a></li>
    <li><a href="https://drive.google.com/file/d/1153MJYi6r2E4fgZCqS7tFwuVYwdstIuJ/view?usp=drive_link">DDL</a></li>
</ul>

<hr>

<h1>🧩 AD – Actividades de Desarrollo</h1>

<!-- AD01 -->
<h2>AD01 – Posibles presentadores</h2>

<h3>Enunciado</h3>
<p>
AD01 - <strong>Posibles presentadores</strong>. Varias veces la empresa se encuentra ante el inconveniente de que un presentador cancele cerca de la fecha de un evento y deben reemplazarlo rápidamente. 
</p>

<p>Para ello crear un procedimiento almacenado <code>posibles_presentadores</code> que reciba una temática, y un rango de fechas y liste los posibles presentadores que:</p>

<ol>
<li>Hayan participado o vayan a participar en el futuro de una presentación en un evento de la temática ingresada</li>
<li>No estén participando de ninguna presentación en ese mismo horario</li>
</ol>

<p>Indicar id, nombre, apellido, denominación y especialidad del presentador y del evento y presentación de las que participó o participará id y nombre del evento y nombre de la presentación.</p>

<p>Probar el procedimiento con la temática "Ciencia Ficcion" para un evento que se organizaría entre 10 de diciembre de 2023 y 21 de diciembre de 2023.</p>

<h3>Resolución</h3>

<pre><code>delimiter
CREATE PROCEDURE posibles_presentadores
(
IN tema  VARCHAR(255),
IN desde DATETIME,
IN hasta DATETIME
)
BEGIN
SELECT DISTINCT
predor.id,
predor.nombre,
predor.apellido,
predor.denominacion,
predor.especialidad,
ev.id,
ev.nombre,
pres.nombre
FROM presentador_presentacion pp
INNER JOIN presentacion pres
ON  pp.id_locacion    = pres.id_locacion
AND pp.nro_sala       = pres.nro_sala
AND pp.fecha_hora_ini = pres.fecha_hora_ini
INNER JOIN evento ev
ON pres.id_evento = ev.id
INNER JOIN presentador predor
ON pp.id_presentador = predor.id
WHERE ev.tematica = tema
AND predor.id NOT IN (
SELECT pp2.id_presentador
FROM presentador_presentacion pp2
INNER JOIN presentacion pres2
ON  pp2.id_locacion    = pres2.id_locacion
AND pp2.nro_sala       = pres2.nro_sala
AND pp2.fecha_hora_ini = pres2.fecha_hora_ini
WHERE pres2.fecha_hora_ini BETWEEN desde AND hasta
OR pres2.fecha_hora_fin  BETWEEN desde AND hasta
);
END

delimiter ;

call posibles_presentadores('Ciencia Ficcion','2023-12-10', '2023-12-21');
</code></pre>

<hr>

<!-- AD02 -->
<h2>AD02 – Salas más pequeñas</h2>

<h3>Enunciado</h3>
<p>
AD02 - <strong>Salas más pequeñas</strong>. La empresa desea ahorrar en el costo de las salas que alquila. 
Para ello se debe identificar para las presentaciones futuras si existe otra sala en la misma 
locación con menor capacidad pero suficiente para las entradas vendidas. 
</p>

<p>
Listar para todas las presentaciones futuras, la capacidad máxima de la sala y 
calcular la cantidad de entradas vendidas. En función de ello determinar si existe una sala diferente en la misma locación, 
que tenga menos capacidad que la sala ya asignada pero suficiente para las entradas vendidas. 
Indicar id y nombre de la locación, número, nombre y capacidad máxima de la sala actual, 
cantidad de entradas vendidas y de las salas potenciales número, nombre y capacidad máxima.
</p>

<h3>Resolución</h3>

<pre><code>with situacion_actual as (
    select pre.id_locacion, pre.nro_sala, pre.fecha_hora_ini,
           sala_actual.capacidad_maxima,
           count(*) entradas_vendidas
    from presentacion pre
    inner join sala sala_actual
        on pre.id_locacion = sala_actual.id_locacion
       and pre.nro_sala = sala_actual.nro
    left join presentacion_entrada pe
        on pre.id_locacion = pe.id_locacion
       and pre.nro_sala = pe.nro_sala
       and pre.fecha_hora_ini = pe.fecha_hora_ini
    where pre.fecha_hora_ini > now()
    group by pre.id_locacion, pre.nro_sala, pre.fecha_hora_ini, sala_actual.capacidad_maxima
)
select sa.id_locacion, loc.nombre as locacion,
       sa.nro_sala as nro_sala_actual,
       sa.capacidad_maxima as capacidad_maxima_actual,
       sa.entradas_vendidas,
       otra_sala.nro as nro_sala_potencial,
       otra_sala.nombre as nombre_sala_potencial,
       otra_sala.capacidad_maxima as capacidad_maxima_potencial
from situacion_actual sa
inner join locacion loc
    on sa.id_locacion = loc.id
left join sala otra_sala
    on sa.id_locacion = otra_sala.id_locacion
   and sa.nro_sala &lt;&gt; otra_sala.nro
   and sa.capacidad_maxima > otra_sala.capacidad_maxima
   and sa.entradas_vendidas &lt;= otra_sala.capacidad_maxima;
</code></pre>

<hr>

<!-- AD03 -->
<h2>AD03 – Registrar cambio en encargados</h2>

<h3>Enunciado</h3>

<p>
AD03 - <strong>Registrar cambio en encargados</strong>. A menudo la empresa debe reasignar los encargados de un evento y luego es difícil saber quienes estuvieron a cargo de un evento durante que período fueron desasignados o saber exactamente cuando y que rol usaron, también desea modificar las condiciones de los encargados ya que en los eventos más grandes y complejos esta siendo difícil la gestión. 
</p>

<p>Por este motivo la empresa necesita:</p>

<ol>
<li>Convertir el rol del encargado en una entidad rol con un id autoincremental y una descripción</li>
<li>Migrar los roles actualmente registrados en encargado_evento a la endidad rol y reemplazar la columna rol por una CF al id de rol.</li>
<li>Registrar en encargado_evento la fecha de asignación obligatoria y fecha fin de asignacion opcional. A los que se encuentran registrados actualmente asígnarles la fecha desde del evento como fecha de asignación y sin fecha de fin.</li>
<li>Ajustar la clave primaria según esta regla: Un empleado puede ser asignado y desasignado a un evento una o varias veces para desempeñar uno o más roles al mismo tiempo o en momentos diferentes y varios empleados pueden desempeñar el mismo rol simultáneamente.</li>
</ol>

<h3>Resolución</h3>

<pre><code>CREATE TABLE `convenciones_underground`.`rol` (
`id` INT NOT NULL AUTO_INCREMENT,
`descripcion` VARCHAR(45) NULL,
PRIMARY KEY (`id`));

ALTER TABLE `convenciones_underground`.`encargado_evento`
ADD COLUMN `idrol` INT NULL AFTER `rol`,
ADD COLUMN `fecha_asignacion` DATETIME NULL AFTER `idrol`,
ADD COLUMN `fecha_fin` DATETIME NULL AFTER `fecha_asignacion`,
ADD INDEX `fk_encargado_evento_rol_idx` (`idrol` ASC) VISIBLE;
;
ALTER TABLE `convenciones_underground`.`encargado_evento`
ADD CONSTRAINT `fk_encargado_evento_rol`
FOREIGN KEY (`idrol`)
REFERENCES `convenciones_underground`.`rol` (`id`)
ON DELETE NO ACTION
ON UPDATE NO ACTION;

BEGIN;
insert into rol (descripcion)
select distinct rol from encargado_evento;

UPDATE encargado_evento ee
INNER JOIN evento ev ON ee.id_evento = [ev.id](http://ev.id/)
INNER JOIN rol r ON ee.rol = r.descripcion
SET ee.fecha_asignacion = ev.fecha_desde,
ee.idrol = [r.id](http://r.id/);

COMMIT;

ALTER TABLE `convenciones_underground`.`encargado_evento`
DROP FOREIGN KEY `fk_encargado_evento_rol`;
ALTER TABLE `convenciones_underground`.`encargado_evento`
DROP COLUMN `rol`,
CHANGE COLUMN `idrol` `idrol` INT NOT NULL ,
CHANGE COLUMN `fecha_asignacion` `fecha_asignacion` DATETIME NOT NULL ,
DROP PRIMARY KEY,
ADD PRIMARY KEY (`id_evento`, `cuil_encargado`, `idrol`, `fecha_asignacion`);
;
ALTER TABLE `convenciones_underground`.`encargado_evento`
ADD CONSTRAINT `fk_encargado_evento_rol`
FOREIGN KEY (`idrol`)
REFERENCES `convenciones_underground`.`rol` (`id`);
</code></pre>

<hr>

<h1>📚 REC – Recuperatorio AD</h1>

<!-- RECAD01 -->
<h2>RECAD01 – Salas alternativas</h2>

<h3>Enunciado</h3>

<p>
RECAD01 - <strong>Salas alternativas</strong>. Cuando una sala tiene un inconveniente cerca de una presentación la empresa debe de forma urgente organizar la presentación en otra sala alternativa. 
</p>

<p>Crear el procedimiento almacenado <code>salas_alternativas</code> que reciba:</p>

<ul>
<li>id de locación</li>
<li>número de sala</li>
<li>rango de fechas donde no podrá utilizarse la sala</li>
</ul>

<p>Y que por cada presentación organizada en ese período devuelva salas que:</p>

<ol>
<li>Sean de la misma locación</li>
<li>Tengan capacidad superior a entradas vendidas</li>
<li>Estén habilitadas</li>
</ol>

<p>Probar con sala 1 de locación 11001 entre 20/11/2023 y 30/11/2023.</p>

<h3>Resolución</h3>

<pre><code>delimiter (T.T)
drop procedure if exists salas_alternativas (T.T)

create procedure salas_alternativas (in loc int unsigned, in sala int unsigned, in desde datetime, in hasta datetime)
begin
with sin_sala as (
    select pre.id_locacion, pre.nro_sala, pre.fecha_hora_ini,
           count(pe.id_evento) entradas
    from presentacion pre
    left join presentacion_entrada pe
        on pre.id_locacion = pe.id_locacion
       and pre.nro_sala = pe.nro_sala
       and pre.fecha_hora_ini = pre.fecha_hora_ini
    where pre.id_locacion = loc
      and pre.nro_sala = sala
      and (
            (pre.fecha_hora_ini >= desde and pre.fecha_hora_ini &lt; hasta)
         or (pre.fecha_hora_fin  >= desde and pre.fecha_hora_fin  &lt; hasta)
          )
    group by pre.id_locacion, pre.nro_sala, pre.fecha_hora_ini
)
select sala.*
from sin_sala
left join sala
    on sin_sala.id_locacion = sala.id_locacion
   and sin_sala.entradas &lt;= sala.capacidad_maxima
   and sin_sala.nro_sala &lt;&gt; sala.nro
where sala.estado = 'habilitada';

end (T.T)

delimiter ;

call salas_alternativas(11001, 1, '2023-11-20','2023-11-30');
</code></pre>

<hr>

<!-- RECAD02 -->
<h2>RECAD02 – Presentaciones a reprogramar</h2>

<h3>Enunciado</h3>

<p>
RECAD02 - <strong>Presentaciones a reprogramar</strong>.  
Indicar la presentación más popular de cada temática que ya se haya finalizado.  
Listar temática, nombre y tipo de la presentación, cantidad de entradas vendidas y nombre del evento donde se realizó.  
Ordenar por cantidad de entradas vendidas descendente y fecha de inicio de la presentación descendente.
</p>

<h3>Resolución</h3>

<pre><code>with max_entradas as (
    select e.tematica, count(*) cant_entradas,
           p.id_evento, p.id_locacion, p.nro_sala, p.fecha_hora_ini
    from evento e
    inner join presentacion p on p.id_evento = e.id
    inner join presentacion_entrada pe
        on pe.id_locacion = p.id_locacion
       and pe.nro_sala = p.nro_sala
       and pe.fecha_hora_ini = p.fecha_hora_ini
    where p.fecha_hora_fin < now()
    group by e.tematica, p.id_evento, p.id_locacion, p.nro_sala, p.fecha_hora_ini
),
max_por_tematica as (
    select tematica, max(cant_entradas) max_entradas
    from max_entradas
    group by tematica
)
select me.tematica, pre.nombre, pre.tipo, e.nombre, me.cant_entradas entradas_vendidas
from max_entradas me
inner join max_por_tematica mt
    on me.tematica = mt.tematica
   and me.cant_entradas = mt.max_entradas
inner join presentacion pre
    on pre.id_locacion = me.id_locacion
   and pre.nro_sala = me.nro_sala
   and pre.fecha_hora_ini = me.fecha_hora_ini 
inner join evento e
    on e.id = me.id_evento
order by me.cant_entradas desc, pre.fecha_hora_ini desc;

</code></pre>

<hr>

<!-- RECAD03 -->
<h2>RECAD03 – Registro de temáticas</h2>

<h3>Enunciado</h3>

<p>
RECAD03 - <strong>Registro de temáticas de evento y presentadores</strong>.  
La empresa notó que la creciente variabilidad en temáticas hace difícil armar presentaciones adecuadas a las temáticas y siempre hay que revisar eventos pasados. 
</p>

<p>Decidió que se estandaricen y registren:</p>

<ol>
<li>Convertir la temática de los eventos en una entidad tematica con un id autoincremental y una descripcion</li>
<li>Migrar las temáticas actualmente registradas en evento a la entidad tematica y reemplazar la columna tematica por una CF al id de temática.</li>
<li>Agregar una relación NaM de presentador a tematica tematica_presentador.</li>
<li>Registrar para cada presentador las temáticas de las que ya ha participado. Para ello revisar las temáticas de los eventos donde realizó una presentación y cargarlas en la tabla tematica_presentador</li>
</ol>

<h3>Resolución</h3>

<pre><code>
CREATE TABLE `convenciones_underground`.`tematica` (
`id` INT NOT NULL AUTO_INCREMENT,
`descripcion` VARCHAR(45) NULL,
PRIMARY KEY (`id`));

---

ALTER TABLE `convenciones_underground`.`evento`
ADD COLUMN `idtematica` INT NULL AFTER `nombre`,
ADD INDEX `fk_evento_tematica_idx` (`idtematica` ASC) VISIBLE;
;
ALTER TABLE `convenciones_underground`.`evento`
ADD CONSTRAINT `fk_evento_tematica`
FOREIGN KEY (`idtematica`)
REFERENCES `convenciones_underground`.`tematica` (`id`)
ON DELETE NO ACTION
ON UPDATE NO ACTION;

---

CREATE TABLE `convenciones_underground`.`tematica_presentador` (
`idtematica` INT NOT NULL,
`idpresentador` INT UNSIGNED NOT NULL,
PRIMARY KEY (`idtematica`, `idpresentador`),
INDEX `fk_tematica_presentador_presentador_idx` (`idpresentador` ASC) VISIBLE,
CONSTRAINT `fk_tematica_presentador_tematica`
FOREIGN KEY (`idtematica`)
REFERENCES `convenciones_underground`.`tematica` (`id`)
ON DELETE NO ACTION
ON UPDATE NO ACTION,
CONSTRAINT `fk_tematica_presentador_presentador`
FOREIGN KEY (`idpresentador`)
REFERENCES `convenciones_underground`.`presentador` (`id`)
ON DELETE NO ACTION
ON UPDATE NO ACTION);

---

BEGIN;

INSERT INTO tematica (descripcion)
SELECT DISTINCT tematica FROM evento;

UPDATE evento ev
INNER JOIN tematica tem ON tem.descripcion = ev.tematica
SET ev.idtematica = [tem.id](http://tem.id/);

INSERT INTO tematica_presentador (idtematica,idpresentador)
SELECT DISTINCT ev.idtematica, pp.id_presentador
FROM presentador_presentacion pp
INNER JOIN presentador pdor ON pdor.id=pp.id_presentador
INNER JOIN presentacion pcion ON pcion.id_locacion = pp.id_locacion AND
pcion.nro_sala = pp.nro_sala AND
pcion.fecha_hora_ini = pp.fecha_hora_ini
INNER JOIN evento ev ON [ev.id](http://ev.id/) = pcion.id_evento;

COMMIT;

---

ALTER TABLE `convenciones_underground`.`evento`
DROP COLUMN `tematica`;
</code></pre>
