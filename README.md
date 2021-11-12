# Testing Spring Boot: Beginner to Guru
Curso de testing
Link: https://www.udemy.com/course/testing-spring-boot-beginner-to-guru/learn/lecture/12076412#questions

## Tabla de Contenidos
- [Testing Spring Boot: Beginner to Guru](#testing-spring-boot-beginner-to-guru)
  - [Tabla de Contenidos](#tabla-de-contenidos)
  - [Conceptos importantes](#conceptos-importantes)
    - [¿Por qué es importante el testing?](#por-qué-es-importante-el-testing)
    - [Unit tests](#unit-tests)
    - [Integration tests](#integration-tests)
    - [Functional tests](#functional-tests)
    - [Errores comunes](#errores-comunes)
    - [Tips](#tips)
    - [Code coverage](#code-coverage)
  - [Test-Driven Development (TDD)](#test-driven-development-tdd)
    - [Ciclo de TDD](#ciclo-de-tdd)
    - [Tips](#tips-1)
  - [JUnit 5](#junit-5)
    - [Configuración](#configuración)
    - [Tips](#tips-2)
    - [Probar las excepciones esperadas](#probar-las-excepciones-esperadas)
    - [Testing timeouts](#testing-timeouts)
    - [Ejecución opcional de tests](#ejecución-opcional-de-tests)
      - [Assumptions](#assumptions)
  - [Testing avanzado (con JUnit)](#testing-avanzado-con-junit)
  - [@RepeatedTest](#repeatedtest)
    - [Casos de uso](#casos-de-uso)
  - [@ParameterizedTest](#parameterizedtest)
    - [Usando un archivo CSV como fuente de datos](#usando-un-archivo-csv-como-fuente-de-datos)
    - [Usando métodos como fuente de datos](#usando-métodos-como-fuente-de-datos)
  - [Tests unitarios vs Tests de Integración](#tests-unitarios-vs-tests-de-integración)
  - [Extensiones de JUnit](#extensiones-de-junit)
  - [Ejecución de Tests](#ejecución-de-tests)
  - [Mockito](#mockito)
    - [Tipos de Mocks](#tipos-de-mocks)
    - [Más terminología](#más-terminología)
    - [Ejemplos Básicos](#ejemplos-básicos)
    - [Inyectando dependencias y retornando valores con *when*](#inyectando-dependencias-y-retornando-valores-con-when)
  - [Behaviour Driven Development (BDD)](#behaviour-driven-development-bdd)
  - [Mockito avanzado](#mockito-avanzado)
    - [Excepciones](#excepciones)
    - [Capturar argumentos con *Argument Captor*](#capturar-argumentos-con-argument-captor)
  - [Dudas](#dudas)
    - [¿Cómo pruebo un método privado?](#cómo-pruebo-un-método-privado)
      - [Usando Java Reflection](#usando-java-reflection)
      - [Usando Springboot](#usando-springboot)
    - [¿Qué sentido tiene testear un Service que lo único que hace es llamar a un Repository?](#qué-sentido-tiene-testear-un-service-que-lo-único-que-hace-es-llamar-a-un-repository)

## Conceptos importantes

### ❔ ¿Por qué es importante el testing?
Ya sé, seguramente estás pensando que la pregunta es estúpida o que sos muy bueno programando y no te hace falta probar tu código porque sabés que va andar. Nada más alejado de la realidad...

Primero que nada, a programar se aprende programando y, ser un **buen programador** es, en gran parte, ser **un programador con buenos hábitos**. Por tanto, si no probás tu código no sos un buen programador.

Segundo, al principio puede que no resulte fácil ver, en términos de costo/beneficio, las ventajas del testing. En especial, el testing sobre el código. Por esta razón voy a empezar nombrando **las desventajas** que trae NO hacer testing apropiadamente:
1. Incertidumbre sobre si un escenario particular está probado y nuestro sistema lo contempla adecuadamente o falla.
2. A medida que crece el proyecto, el mismo se vuelve más complejo y dificil de mantener.
3. El crecimiento acelerado y con un equipo trabajando en simultáneo, sin el testing adecuado es propenso a errores y a bajar la calidad del código.
4. Aunque seamos muy cuidadosos, puede que un compañero nuestro añada una funcionalidad que entra en conflicto con la nuestra y nos percatemos del error hasta que nos explota en la cara.
5. Baja calidad del código: escribir tests no solo nos asegura que nuestro código está probado sino que nos fuerza a pensar y escribir nuestro código de manera que podamos testearlo, y en general, esto conduce a la generación de código más sólido.
6. Tests de regresión manuales: añadimos un cambio en nuestro código pero debemos asegurarnos que sea *retrocompatible* y no rompa escenarios anteriores. Sino tenemos tests de código automátizados, debemos  que ejecutar *tests funcionales (TD y TE)* manualmente. Esto hace que el **tiempo que nos lleva probar nuestro sistema aumente exponencialmente**.
7. Un cambio menor hace un desastre: muchas veces terminamos nuestro desarrollo, comenzamos con las pruebas y durante ellas detectamos un pequeño error que decimos "Ah, esto lo soluciono en un segundo con 2 líneas de código" y realizamos el cambio pero no lo probamos porque "no hace falta". Luego nos enteramos del error que introdujimos cuando ya es tarde...
8. Miedo al refactor: tengo que hacer una modificación en una parte del código que del cual dependen muchas otras funciones y temo olvidar contemplar algún caso e introducir un bug.

El **punto 3** suele ser mitigado usando **branching** usando *GIT* en lugar de *SVN* y realizando una integración/merge manual de los distintas funcionalidades desarrolladas por los programadores.

Para evitar el **punto 8** a veces terminamos creando un nuevo método especificamente para el cambio que debo introducir, quedando así 2 métodos (el original y el nuevo) **con una misma responsabilidad, seguramente con código duplicado** y que, el día de mañana cuando volvamos a leerlo, no entendemos por qué razón está hecho así. Esto es un **Anti-patrón**.

También nombro algunas **ventajas** del testing:
1. Los tests de código sirven como documentación del mismo. Sabemos concretamente que escenarios están cubiertos y cuales no.


Hay varios tipos de tests:

### ✔️ Unit tests
* Son muy chicos, están diseñados para probar una parte específica del código y son los que más abundan. 
* No deben llamar dependencias ni levantar spring context, h2 o consultar un WebService (esto ya sería Integration Tests), ya que deben ser super rapidos. 
* Si pueden usar MockUps.
* Deben testear entre un 70-80% del código. 
* La lógica del negocio debería probarse aquí dice el Guru. (Me parece raro, yo imaginé que sería en los tests de Integración, pero tal vez debamos pasarle los inputs necesarios para simular el contexto al test unitario)

### ✅ Integration tests
* Tienen un scope más grande que los tests unitarios.
* Pueden incluir Spring Context, consultas a BD u otros servicios.
* Consecuentemente son mucho más lentos.
* En la proporción hay mucha menor cantidad.

### 🧑‍🦯 Functional tests
* Es cuando estás probando la aplicación andando en algún ambiente, no una parte del código.
* Pueden ser automatizados con frameworks de automatización como Selenium.
* Generalmente se prueban funcionalidades, por ej: WebServices, enviar y recibir mensajes, procesamiento de información, etc.
* En la proporción, conforman el menor porcentaje de los tests ya que son comparativamente muy lentos.
* Deberíamos usarlos solo para probar funcionalidad crítica.

### ❌ Errores comunes
* Mezclar Unit Tests con Integration Tests.
* Muchos tests levantando Spring Context: el build se vuelve lentísimo.
* Un test unitario no debería preocuparse por levantar el contexto necesario para ejecutar un escenario, sino solo que un método devuelva una cierta salida dadas unas condiciones de entrada. Es decir, no prueba un escenario con DATOS REALES sino que verifica que una función hace su trabajo.
* En cambio un Test de integración SI prueba un escenario. Levanta el contexto/configuración y carga datos de un escenario real que debería poder llevarse a cabo.

### 💡 Tips
* "I'm not a great programmer. I'm just a good programmer with great habits" - Kent Beck.
* Si hay tests que pueden hacen fallar a la aplicación una vez que el desarrollo de una funcionalidad está finalizado, entonces esos tests deberían formar parte del ciclo de desarrollo.

### Code coverage
Es el porcentaje de líneas de código que son testeadas


## Test-Driven Development (TDD)
Consiste en primero escribir el Test y luego desarrollar el código.
El Test se escribe directamente cuando ni siquiera las clases han sido creadas, por lo que estará "lleno de errores".
La idea es que a medida que implementamos la funcionalidad el Test debería "corregirse" solo y cuando hayamos finalizado la implementación, el test debería dar OK.

**Objetivo**: "Clean code that Works".

### Ciclo de TDD
1. **Escribir el test**: pensar como debería funcionar (diseñar).
2. **Hacer que cumpla el test**: Implementar *rapidamente* el código que pasa el test.
3. **Hacerlo bien**: Refactorizar el código mejorando la calidad.

Podemos usar una *TO-DO List* para poner items que la implementaicón debe cumplir. Estas a su vez, se transformarán en Tests que deberá pasar.

Si aún sentis que no entendiste TDD al 100%, cosa que es completamente normal, te recomiendo fuertemente leer [este artículo](http://www.jamesshore.com/v2/blog/2005/red-green-refactor) que hizo *James Shore* en donde lo explica **más cercano a la realidad y fácil de entender**.

### :bulb: Tips
* Se puede más de una assertion por test.
* Puede ser útil poner un assertEquals y un assertNotEquals para contemplar más casos.
* equals() por defecto compara las ubicaciones de memoria, pero generalmente vamos a querer sobreescribirlo para saber si dos objetos son **conceptualmente** iguales. EJ: 2 personas con el mismo DNI, son iguales.
* Mejor que borrar un test porque ya no se usa, es deshabilitarlo comentando la razón. Ejemplo:
```java
@Test
@Ignore("Null values no longer allowed")
public void testDistinctOfSourceWithExceptionsFromKeySelector() 
```


## JUnit 5
### Configuración
* Recomienda **maven surefire** 2.22.0 o superior! Este plugin se encarga de ejecutar los tests unitarios mediante Maven. Lo mismo para **maven-failsafe-plugin**.
* Para manejar las versiones pone dentro del <properties> del *pom.xml* una property llamada <junit-plataform.version> y luego al invoca como variable `$junit-plataform.version` en el campo *version* de las otras dependencias.
* A las dependencias de JUnit les pone la property <scope>test</scope>. Esto hace que los tests no se compilan dentro del .jar. [Mas info: Maven Scopes](https://www.baeldung.com/maven-dependency-scopes)
* 

### Tips
* Podemos crear un metodo *setUp* con @BeforeEach que se ejecuta **antes de cada test** de esa clase e inicializar Objetos y variables que se usan dentro de los Tests. Viene a ser como una especie de "contexto" que se levanta con cada clase de Tests. Algunas ventajas son:
  * Evita duplicar código.
  * Util para levantar Mocks.
  * Aumenta la velocidad de ejecucion de los tests.
  * Los objetos estos deben ser atributos de clase, no variables internas al método *setUp*.
* Hay mas anotaciones de este estilo, las cuales forman el *ciclo de vida de JUnit*:
```java
@BeforeAll public static void beforeClass(){} // Solo se ejecuta una vez antes de todos los Tests
    @BeforeEach void setUp(){}
        @Test void myFirstTest(){}
    @AfterEach void tearDown(){} 
@AfterAll void afterClass(){} // Solo se ejecuta una vez al final
```
* Hay FRAMEWORKS especializados en *assertions*. Por ejemplo, para manejar JSONs:
  * **AssertJ**: agrega funcionalidad muy copada, por ejemplo, contiene una funcion `isXmlEqualToContentOf(File f)` que permite comparar un XML generado contra un ejemplo fijo. Esto puede resultar muy util para los casos de prueba contra JDE.
  * Hamcrest
  * Truth
* Hay algunas assertions especiales:
  * `fail()` --> Para asegurar que algo no se rompa podemos hacer if (condicion) {fail()}
  * `assertThrows()` o **assertDoesNotThrow()** --> Para asegurar que se están manejando bien las excepciones
  * `assertTimeout()` --> Para asegurar que un método tarde menos de un cierto tiempo. No es muy usado ya que esto puede variar por muchos factores.
* `@Disabled(value = "Necesitamos hacer un mock para probar esto")`
* Podemos usar `@DisplayName("Debe tirar una excepcion si el tipo de cambio es negativo")` para mostrar un nombre amistoso

### Probar las excepciones esperadas
Muchas veces solo testeamos el camino feliz, pero es necesario probar también que estamos controlando los errores/excepciones adecuadamente.
Para esto está el método `assertThrows(ExampleExcepction.class, () -> {Código que produce la excepción})`

### Testing timeouts
Probar los tiempos sirve para varias cosas, entre ellas, para asegurar que no hemos metido ningún bug de performance.

**Es importante tener en cuenta que los tiempos pueden variar de un entorno al otro y quizás un test que pasaba en nuestra pc, no pasa en el ambiente de CI.**
Para estos podemos usar 2 metodos:
```java
assertTimeout() // Ejecuta el test y una vez que terminó se fija si excedió el tiempo estipulado o no.
assertTimeoutPreemptively() // Este ejecuta el test en otro hilo y aborta su ejecución una vez que se superó el timeout
```

### Ejecución opcional de tests
Podemos hacer que los tests se ejecuten solo bajo algunas condiciones. Hay varias formas de hacerlo:
* Assumptions
* Annotations
```java
@EnabledOnOS
@EnabledOnJre
@EnabledIfEnvironmentVariable(name = "USER", matches = "juancruz") // Esta es la más utilizada
// etc
```

#### Assumptions
Son presunciones. Nos permite evaluar si se cumple una condición antes de ejecutar un test. Generalmente son útiles para evaluar variables de entorno y/o configuraciones del ambiente en el que se está corriendo el test y en base a eso, decidir si se ejecuta un test o no.

Dado que permiten decidir si se ejecuta o no, generalmente son las primeras lineas de los tests unitarios **aunque esto no es necesario**. Es posible tener más de una assumption.

Si no se cumple la condición el test será marcado como **ignorado**, no como fallado.
`assumeTrue("TEST".equals(System.getenv("env")))`

## Testing avanzado (con JUnit)
* Se puede usar `@Tag` y `@DisplayName` para mejorar el nombre visible del test. **Tag** sirve basicamente para agrupar Tests. Luego podemos decirle a a IntelliJ IDEA que ejecute *todos los tests con el tag "Controllers"* desde *Launch configurations*.
* `@Nested` permite definir una clase de Test dentro de otra. Sirve para agrupar Tests.
* Interfaces para los Tests? Al pedo.
* `@TestInstance(TestInstance.LifeCycle.PER_CLASS` Se utiliza a nivel de clase para definir el scope del `@BeforeAll`

## @RepeatedTest
`@RepeatedTest(value=10, name="{displayName}: {currentRepetition} of {totalRepetitions}")` nos permite hacer que un test se ejecute **n** veces seguidas, ajustando el parámetro **value**. Por otro lado, **name** es el formato en el que se muestran las ejecuciones de los tests. Cuando usamos esta annotation no hace falta agregar `@Test` al método.

También pueden usarse unos parámetros que nos brindan información extra. Estos son: ***TestInfo***, ***RepetitionInfo*** y ***TestReporter***.

### Casos de uso
* Imagino que es útil cuando querés probar que una cierta funcionalidad **mantiene un estado** y **cumple con las restricciones impuestas**. Por ejemplo, *"un Club puede tener como máximo 2 miembros"*:
```java
// Instancio algún objeto fuera del TestUnitario (por ej, en @BeforeAll) para que la instancia mantenga el estado entre un test y el otro

@RepetatedTest(3)
private void whenMembersGreaterThan2_expectClubIsFullException(RepetitionInfo repetitionInfo) {
    if (club.getMembersCount() < 2) {
        int expected = club.getMembersCount() + 1;
        club.addMember(new Member("Juan"));
        assertTrue(club.getMembersCount(), expected);
    } else {
        MyException thrown = assertThrows(
            MyException.class,
            () -> club.addMember(new Member("Juan"))
        );

        assertTrue(thrown.getMessage().contains("Stuff"));
    }
}
// Si bien en este caso no usé repetitionInfo, podría imaginar un caso parecido a este.
```

## @ParameterizedTest
Es parecedio a `@RepeatedTest` ya que repite el test por cada conjunto de parámetros que le pasamos.

```java
@ParameterizedTest
@ValueSource(strings = {"Juan", "Cruz"})
void testValueSource(String value) {
    System.out.println(value);
}

@ParameterizedTest
@EnumSource(CarType.class)
void testEnumSource(CarType carType) {
    // Corre el test una vez por cada Enum que esté definido en la clase CarType
    System.out.println(carType);
}

public class enum {
  	SPORT, DELUXE;
}
```

### Usando un archivo CSV como fuente de datos
Corre el test una vez por cada tupla en el archivo csv que se encuentra en la carpeta `resources/input.csv`.

El parámetro `numLinesToSkip` evita que lea el header del csv como una tupla

```java
@ParameterizedTest
@CsvFileSource(resources = "/input.csv", numLinesToSkip = 1)
void testCsvFileSource(String stateName, int val1, int val2) {
    System.out.println(stateName + val1 + val2);
}
```

### Usando métodos como fuente de datos
Corre el test una vez por cada tupla que devuelva el método `getargs()` definido a continuación.

Esto nos da la ventaja de poder obtener los parámetros de alguna forma no "estandarizada" por JUnit, por ejemplo, desde la BD, desde un JSON, un XML, etc.

Además lo podemos reutilizar en distintos tests.

```java
@DisplayName("Method Provider Test")
@ParameterizedTest(name = "{displayName} - [{index}] {arguments}")
@MethodSource("getargs")
void fromMethodTest(String stateName, int val1, int val2) {
    System.out.println(stateName + val1 + val2);
}

static Stream<Arguments> getargs() {
    return Stream.of(
            	Arguments.of("AR", 1, 1),
            	Arguments.of("BR", 2, 3)
    );
}
```

También podemos externalizar los datos de prueba a una clase utilizando `@ArgumentsSource`. No tengo en claro en qué escenarios resultaría cómodo esto, pero quizás sea cuando necesitamos agrupar un gran número de argumentos para representar un escenario en particular.

```java
@DisplayName("Custom Provider Test")
@ParameterizedTest(name = "{displayName} - [{index}] {arguments}")
@ArgumentsSource(CustomArgsProvider.class)
void fromCustomProviderTest(String stateName, int val1, int val2) {
    System.out.println(stateName + val1 + val2);
}

public class CustomArgsProvider implements ArgumentsProvider {
    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext extContext) throws Exception {
        return Stream.of(
            	Arguments.of("AR", 1, 1),
            	Arguments.of("BR", 2, 3)
        );
    }
}
```

## Tests unitarios vs Tests de Integración
**Maven** usa con convención en el nombre de las clases de test para saber cuál es un **Test de integración**. Para detecte nuestra clase de test como tal, debemos nombrarla con el sufijo **IT** (que viene de *Integration Test*). Por ejemplo: `UsersServiceIT.java`

## Extensiones de JUnit
Hay una librería propia de JUnit que ofrece extensiones básicas como `BeforeTestExecutionCallback` y `AfterTestExecutionCallback` que pueden resultar útiles para medir los tiempos de un Test.
*Spring* y *Mockito* usan estas extensiones para agregar funcionalidad.

```java
import guru.springframework.sfgpetclinic.junitextensions.TimingExtension;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(TimingExtension.class)
class PetTypeSDJpaServiceIT {
    @Test
    void findAll() {
    }
}


import org.junit.jupiter.api.extension.AfterTestExecutionCallback;
import org.junit.jupiter.api.extension.BeforeTestExecutionCallback;
import org.junit.jupiter.api.extension.ExtensionContext;

/**
 * Original source - https://junit.org/junit5/docs/current/user-guide/#extensions-lifecycle-callbacks-timing-extension
 */
public class TimingExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private static final Logger logger = Logger.getLogger(TimingExtension.class.getName());

    private static final String START_TIME = "start time";

    @Override
    public void beforeTestExecution(ExtensionContext context) throws Exception {
        getStore(context).put(START_TIME, System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        Method testMethod = context.getRequiredTestMethod();
        long startTime = getStore(context).remove(START_TIME, long.class);
        long duration = System.currentTimeMillis() - startTime;

        logger.info(() -> String.format("Method [%s] took %s ms.", testMethod.getName(), duration));
    }

    private ExtensionContext.Store getStore(ExtensionContext context) {
        return context.getStore(ExtensionContext.Namespace.create(getClass(), context.getRequiredTestMethod()));
    }
}
```

## Ejecución de Tests
* Se puede limitar la cantidad de tests con *Surefire* poniendo en `<group>` los **Tags** del grupo de tests que queremos ejecutar y/o excluir.
* **CircleCI** es el framework de *Continuous Integration* utilizado por el chabon del curso.


## Mockito
* Es el framework de testing mas popular para "mockear" o simular objetos.
* Está pensado para reemplazar a los objetos reales con objetos de prueba en los tests unitarios.
* Funciona bien con **inyección de dependencias** (y por lo tanto con Spring), **evitándo tener que levantar *Spring Context*** (lo que convertiría al test unitario en un "test de integración").
* Página oficial: [https://site.mockito.org/](https://site.mockito.org/)
* Artículo hecho por los autores: [¿Cómo escribir buenos tests con Mockito?](https://github.com/mockito/mockito/wiki/How-to-write-good-tests)

### Tipos de Mocks
Se definen diferentes tipos de mocks:
1. **Dummy:** se usa solo para que el código compile pero nunca es llamado. El típico caso donde una función tiene más argumentos de los que vamos a testear.
2. **Fake:** es un objeto que tiene una implementación pero no se usa en producción. **¿Ejemplo?**
3. **Stub:** un objeto con respuestas pre-definidas en sus métodos. Por ejemplo, una instancia *Person* en particular.
4. **Mock:** un objeto con respuestas pre-definidas en sus métodos pero además puede tener lógica implementada para simular comportamiento e incluso lanzar excepciones si se produce algo ilógico.
5. **Spy:** es un wrapper para el Mock que permite interceptar las llamadas al mock e inspeccionar el estado del objeto.

### Más terminología
* **Verify:** se usa para verificar el numero de veces que el método de un mock ha sido llamado.
* **Argument matcher:** no explica como se usa. **COMPLETAR**
* **Argument captor:** permite interceptar los argumentos del método del mock según su valor. Esto nos permite agregar lógica al método del mock basado en los argumentos.

### Ejemplos Básicos
```java
public class MockTest {

	@Test
	void testInlineMock() {
		Map mapMock = mock(Map.class); // Inicialización de un mock por defecto
		assertEquals(mapMock.size(), 0);
	}


	@Mock
	Map<String, Object> mapMock2;

	@BeforeEach
	void setUp() {
		MockitoAnnotations.initMocks(this); // Inicializa todos los mocks indicados con @Mock
	}

	@Test
	void testAnnotatedMock() {
		assertEquals(mapMock2.size(), 0);
	}
}

// MockitoExtension corre un BeforeEach inicializando los mocks, por lo tanto nos ahorra el trabajo de escribirlo en cada test
@ExtendWith(MockitoExtension.class)
public class MockitoExtensionTest {
	@Mock
	Map<String, Object> mapMock2;

	@Test
	void testAnnotatedMock() {
		assertEquals(mapMock2.size(), 0);
	}
}
```

### Inyectando dependencias y retornando valores con *when*
En el siguiente ejemplo vamos a ver como levanto un service que depende de un repository y como podemos inyectar las dependencias en el service con Mockito. 

Otra ventaja de estos mocks es que **no levantan el contexto de spring**, lo que hace que *"sigan siendo tests unitarios"* super rápidos.

En la función `findByIdTest()` utilizo la función `when()` y `thenReturn()` para indicar el valor que quiero retornar cuando una función sea llamada. Y esta función pertenece, justamente, a la dependencia que acabo de mockear con `@Mock` *"SpecialtyRepository"* la cual al ser un Mock, no sabe qué responder hasta que yo se lo indico de esta manera.

```java
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;

@ExtendWith(MockitoExtension.class)
public class SpecialitySDJpaServiceTest {

    @Mock
    SpecialtyRepository specialtyRepository;

    @InjectMocks // Inyecta los @Mocks necesarios en este service para sastisfacer las dependencias (algo que normalmente hace spring)
    SpecialitySDJpaService service;

    @Test
    void delete() {
        Speciality speciality = new Speciality();
        service.delete(speciality);
        // Verifica que se llamó solo 1 vez el método delete en el repository.
        verify(specialtyRepository, times(1)).delete(speciality);
    }

    @Test
    void deleteById() {
        Speciality speciality = new Speciality();
        service.deleteById(speciality.getId());
    }

    @Test
    void findByIdTest() {
        Speciality speciality = new Speciality();
        when(specialtyRepository.findById(1L)).thenReturn(Optional.of(speciality)); // Defino el comportamiento del Mock
        Speciality foundSpeciality = specialtyRepository.findById(1L);
        assertThat(foundSpeciality).isNotNull();
        verify(specialtyRepository).findById(1L); // Verifica que se llamó solo 1 vez el método findById.
    }
}
```

Como habremos notado, la función `verify()` de *Mockito* no parece realmente servir de algo al verificar que realmente se llamó al *Repository* luego de hacer el `delete`. Además, al usarlo estás "atando" el test a _la implementación particular del repository_ que estás usando. Y esto **puede que te obligue a corregir** esa parte del test si haces un *refactor* sobre el repository. Es decir, agrega un overhead.

En otros escenarios donde sí nos interesa hacer una verificación más minuciosa, espcialmente en algún ***Test de integración***, este método si nos puede resultar útil. Dejo el link a [este thread](https://stackoverflow.com/questions/12539365/when-to-use-mockito-verify) de StackOverflow en donde justamente se discute esto y explican más detalladamente lo que acabo de decir.



## Behaviour Driven Development (BDD)
Este enfoque se centra en que **los tests prueben los escenarios posibles** y no que el código sea necesariamente super-robusto. Es decir, lo opuesto a **property-based testing (PBT)**.
Considera a los tests unitarios como *"la especificación del comportamiento del programa"*.

Para definir los escenarios propone usar la keywords *given, when, then*, y los nombres de lo métodos de test deben ser oraciones, ej `saveValidPerson()`.
Acá muestro dos ejemplos equivalentes (el segundo está escrito en BDD), pero la funcionalidad es la misma:
```java
@ExtendWith(MockitoExtension.class)
public class SpecialitySDJpaServiceTest {
    @Test
    void findByIdTest() {
        Speciality speciality = new Speciality();
        when(specialtyRepository.findById(1L)).thenReturn(Optional.of(speciality));
        Speciality foundSpeciality = specialtyRepository.findById(1L);
        assertThat(foundSpeciality).isNotNull();
        verify(specialtyRepository).findById(1L);
    }

    @Test
    void findByIdTest() { // BDD
        // given
        Speciality speciality = new Speciality();
        given(specialtyRepository.findById(1L)).willReturn(Optional.of(speciality)); 
        
        // when
        Speciality foundSpeciality = specialtyRepository.findById(1L);
        
        // then
        assertThat(foundSpeciality).isNotNull();
        then(specialtyRepository).should(times(1)).findById(1L);
    }
}
```

Como vemos, BDD es solo una cuestión de enfoque y nomenclatura, no va más allá de ello. Personalmente creo que es útil tener tanto tests que verifiquen que un en método en particular funcione coherentemente (sin pensar en si se va a usar en ese escenario o no), como tests que verifiquen los escenarios o "la lógica del negocio".

Además, a menudo nos encontramos con que no podemos probar un método de manera aislada porque está como privado (`private`), sin embargo estaremos probándolo si cubrimos todos los escenarios posibles para una funcionalidad "padre" la cual consume este método privado. (Para más info, ver nota al final)

## Mockito avanzado

### Excepciones
En el curso da un ejemplo donde mockea un *Repository* para que lanze una excepción en un determinado caso. Si bien esto no parece tener demasiado sentido, debemos recordar que cuando mockeamos un objeto es porque necesitamos probar otra funcionalidad que depende en parte del objeto mockeado. Entonces **podemos mockear una excepción si, por ejemplo, queremos ver si una porción del código está atajando bien las excepciones.**

Un ejemplo concreto que se me ocurre podría ser:
> Supongamos que tenemos 10 productos en stock y quiero vender 50. 
> 
> Debería testear que en caso de que esto ocurra, se lanza una excepción de que no hay suficiente stock **y que esta excepción es correctamente capturada por quién hizo la llamada al método**, mostrando un mensaje amistoso para el usuario.

Podemos probar esto **mockeando la excepción de que no hay suficiente stock** y hacer un test para verificar que capturamos y tratamos bien la excepción.

*Mockito* soporta esto tanto para **TDD** como para **BDD**.

### Capturar argumentos con *Argument Captor*
¿Para que sirve?
¿Cómo se usa?


## ❓ Dudas 
* ¿Cuándo es útil usar `@RepeatedTest`?
* ¿Cuándo es útil usar `@ArgumensProvider`?
* ¿Cuándo es útil usar **Arguments Matcher**?
* No entiendo para qué es el Maven Surefire Plugin. ¿Hace que aparezca la opción test en el ciclo de vida maven?
* Tampoco el Maven Failsafe Plugin. Por lo que veo, sin esos plug-ins, no se VE en la consola la corrida de los tests. Ambos son para generar reportes. Por alguna razón los Tests de Integración los ejecuta con este plugin en **mvn verify** poniendo como **goals** `<goals><goal>integration-test<goal><goal>verify<goal><goals>`. Estos plugins se usan junto con maven-site plugin y configurando la seccion `<reporting>` del *pom.xml*.

### ✔️ ¿Cómo pruebo un método privado?
Hay varios enfoques interesantes en cuanto a esto:

**[Algunos dicen](https://stackoverflow.com/a/34586)** que la mejor forma es probar un método privado es a través de otro método publico que lo llame. Y que si esto no se puede hacer, se da una de las siguientes condiciones:
1. El método privado es código muerto (nunca se utiliza).
2. Hay un problema de diseño en la clase que estás testeando.
3. El método no debería ser privado. **ACLARACION: Un método nunca debería ser forzado a ser público con el único fin de poder testearlo.**

Sin embargo, si consideramos que el método privado amerita un testing más exhaustivo, podemos valernos de algunas herramientas para ejecutar ese método privado **sin convertirlo en público**:

#### Usando Java Reflection
```java
Method method = TargetClass.getDeclaredMethod(methodName, argClasses);
method.setAccessible(true);
return method.invoke(targetObject, argObjects);

// Para atributos de clase privados
Field field = TargetClass.getDeclaredField(fieldName);
field.setAccessible(true);
field.set(object, value);
```

#### Usando Springboot
```java
import org.springframework.test.util.ReflectionTestUtils;

// ...

@Test
    public void whenNoAfipValidation_thenAddWarning() {
        ArrayList<ValidationProcess> validationProcesses = new ArrayList<>();
        validationProcesses.add(new ValidationProcess("TOTAL", "", 0, true, "com.besysoft.validationsmanagement.validationprocess.CalculateTotalValidation"));
        Object result = ReflectionTestUtils.invokeMethod(service, "hasAnyAfipValidation", validationProcesses);
        if (result instanceof Boolean) {
            boolean hasAfipValidation = (boolean) result;
            assertEquals(false, result);
        } else
            fail();
    }

```

También existe una librería **[PowerMock](https://www.baeldung.com/powermock-private-method)** que permite hacerlo de una manera similar a las 2 anteriores. De todas maneras, recalco que en mi opinion, no se deben hacer tests sobre métodos privados exceptos casos puntuales.

### ✔️ ¿Qué sentido tiene testear un Service que lo único que hace es llamar a un Repository?
Es la pregunta que siempre me hice. Uno de los instructores responde pero no da un escenario de la vida real:
()[https://www.udemy.com/course/testing-spring-boot-beginner-to-guru/learn/lecture/12562476#questions/5700521]

"La verdad es que se usa solo en TDD, ya que se escribe primero el test y luego el código."
