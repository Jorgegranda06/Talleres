Estudiantes.csv
nombre,edad,calificacion,genero
Andrés,10,20,M
Ana,11,19,F
Luis,9,18,M
Cecilia,9,18,F
Katy,11,15,F
Jorge,8,17,M
Rosario,11,18,F
Nieves,10,20,F
Pablo,9,19,M
Daniel,10,20,M

Case Class
```Scala
package models

case class Estudiante(
                       id: Int,
                       nombre: String,
                       edad: Int,
                       calificacion: Int,
```
Object
```Scala
package Clase15

import doobie._
import doobie.implicits._
import cats.effect._
import cats.implicits._
import scala.io.Source
import models.Estudiante

object TallerReto extends IOApp.Simple {

  // Transactor a la base de datos
  val transactorDB = Transactor.fromDriverManager[IO](
    driver = "com.mysql.cj.jdbc.Driver",
    url = "jdbc:mysql://localhost:3306/estudiantes_db",
    user = "root",
    password = "Jorge2010dome@",
    logHandler = None
  )

  // Inserta un estudiante
  def guardarEstudiante(nombre: String, edad: Int, nota: Int, genero: String): ConnectionIO[Int] =
    sql"""
      INSERT INTO estudiantes (nombre, edad, calificacion, genero)
      VALUES ($nombre, $edad, $nota, $genero)
    """.update.run

  // Lista todos los estudiantes
  def obtenerEstudiantes(): ConnectionIO[List[Estudiante]] =
    sql"""
      SELECT id, nombre, edad, calificacion, genero
      FROM estudiantes
    """.query[Estudiante].to[List]

  // Limpia la tabla
  def vaciarTabla(): ConnectionIO[Int] =
    sql"TRUNCATE TABLE estudiantes".update.run

  override def run: IO[Unit] = {

    val filasCsv = Source
      .fromFile("C:\\Users\\Core i5-Pro\\IdeaProjects\\Clasejueves4\\src\\main\\resources\\data\\estudiantes.csv")
      .getLines()
      .drop(1)
      .toList

    val inserciones = filasCsv.map { fila =>
      val datos = fila.split(",")
      guardarEstudiante(
        datos(0).trim,
        datos(1).trim.toInt,
        datos(2).trim.toInt,
        datos(3).trim
      )
    }

    val programaDB = for {
      _ <- vaciarTabla()
      _ <- inserciones.sequence
      lista <- obtenerEstudiantes()
    } yield lista

    programaDB.transact(transactorDB).flatMap { estudiantes =>
      IO.println("\n--- LISTADO DE ESTUDIANTES ---") *>
        IO {
          estudiantes.foreach { e =>
            println(f"${e.id}%2d | ${e.nombre}%-10s | ${e.edad}%2d años | ${e.calificacion}%2d | ${e.genero}")
          }
        } *>
        IO.println(s"\nTotal de estudiantes registrados: ${estudiantes.size}")
    }
  }
}
```
                       genero: String
                     )
