# Actividad 1: Identificación y Corrección de Infracciones a los Principios SOLID 
*Curso:* Programación Orientada a Objetos II (POO II)  
*Integrantes*
CAJA CORRALES, DAIN JAIR 
RETES GUTIERREZ, PIERO FERNANDO 
SOSA TACO, CELINA OLENKA 
YURIVILCA LAUREANO, STEFFI IRENE

## Objetivo de la Actividad
El propósito de esta actividad es analizar un código fuente heredado (legacy) que infringe las reglas del diseño orientado a objetos, identificar de manera crítica las violaciones a los principios *SOLID* y proponer e implementar una solución robusta y flexible en lenguaje Java que cumpla con los estándares de la ingeniería de software moderna.

##  1. Análisis de Infracciones al Diseño de Software

En la siguiente tabla se detallan los hallazgos críticos detectados en el diseño original del sistema de gestión de empleados:

| Principio SOLID | Estado en Código Original | Justificación Técnica y Consecuencias |
| :--- | :--- | :--- |
| *S* - Single Responsibility Principle (Responsabilidad Única) | ❌ *Violado* | La clase Empleado mezcla el *"Ser" con el "Hacer"* (su estado como entidad de datos y procesos de negocio/infraestructura) [5, 6]. Al encargarse de almacenar datos, calcular nóminas, gestionar la persistencia en base de datos y dar formato a reportes, asume múltiples razones para cambiar [7-9]. |
| *O* - Open/Closed Principle (Abierto para Extensión, Cerrado para Modificación) | ❌ *Violado* | El método calcularPago() depende de una estructura condicional rígida (if/else if) basada en un atributo de tipo cadena (tipo.equals(...)) [7]. Agregar un nuevo rol obliga a modificar directamente el código interno de la clase, lo cual puede introducir regresiones [8, 10, 11]. |
| *L* - Liskov Substitution Principle (Sustitución de Liskov) | ❌ *Violado* | El sistema no emplea herencia real ni abstracciones [1, 7]. La lógica de cálculo depende de la validación condicional de cadenas de texto [7]. Esto impide que los diferentes roles se comporten como subtipos polimórficos de una base común de manera transparente en tiempo de ejecución [10, 12]. |
| *I* - Interface Segregation Principle (Segregación de Interfaces) | ⚠️ *No Aplicado* | Al no definirse contratos abstractos, el sistema fuerza a que las funcionalidades de persistencia y reportería se acoplen directamente en la clase Empleado [7]. Un cliente que solo requiera leer datos se ve forzado a depender de métodos de infraestructura de escritura [8, 10, 13]. |
| *D* - Dependency Inversion Principle (Inversión de Dependencia) | ❌ *Violado* | La clase controladora de alto nivel (SistemaGestionEmpleados) depende de implementaciones concretas de bajo nivel (Empleado) [1, 8, 14]. No existen abstracciones (interfaces) que desacoplen las reglas del negocio de los detalles de infraestructura o persistencia [8, 14, 15]. |



## 🛠️ 2. Arquitectura de la Solución Propuesta

Para corregir estas infracciones, se diseñó un modelo de software bajo las siguientes pautas:

1. *Entidades Claras (SRP):* La clase abstracta Empleado ahora solo contiene los datos esenciales del colaborador [5, 6]. Las responsabilidades de almacenamiento y visualización se extrajeron a la capa de infraestructura [5, 6].
2. *Polimorfismo para Extensibilidad (OCP/LSP):* Empleado se define como una clase abstracta con un método abstracto calcularPago() [5, 16]. Cada rol (Gerente, Desarrollador y Practicante) hereda de esta y define su propia lógica de nómina, permitiendo añadir nuevos tipos de empleados sin modificar el código existente [12, 16, 17].
3. *Inyección de Dependencias por Abstracción (DIP/ISP):* Se crearon las interfaces EmpleadoRepository y ReporteService [8, 14, 18]. El controlador SistemaGestionEmpleados se comunica únicamente con estas interfaces abstractas [14, 15, 19]. Las dependencias concretas son inyectadas a través de su constructor [19].



## 💻 3. Código Fuente Refactorizado (Java)

El siguiente código fuente implementa la solución propuesta aplicando de forma estricta los principios SOLID:

```java
// ==========================================
// 1. CAPA DE DOMINIO (ABSTRACCIÓN Y ENTIDADES)
// ==========================================

/**
 * Representa a un Empleado.
 * Cumple con SRP al limitarse a los datos de la entidad de dominio.
 * Cumple con OCP al permitir la extensión polimórfica de roles.
 */
abstract class Empleado {
    private String nombre;
    protected double salario;

    public Empleado(String nombre, double salario) {
        this.nombre = nombre;
        this.salario = salario;
    }

    public String getNombre() {
        return nombre;
    }

    public double getSalario() {
        return salario;
    }

    // Método abstracto para cálculo de pago (Polimorfismo)
    public abstract double calcularPago();
}

/**
 * Subclase Gerente (Sustituye a Empleado aplicando LSP)
 */
class Gerente extends Empleado {
    public Gerente(String nombre, double salario) {
        super(nombre, salario);
    }

    @Override
    public double calcularPago() {
        return salario + 1000; // Bono de gerencia
    }
}

/**
 * Subclase Desarrollador (Sustituye a Empleado aplicando LSP)
 */
class Desarrollador extends Empleado {
    public Desarrollador(String nombre, double salario) {
        super(nombre, salario);
    }

    @Override
    public double calcularPago() {
        return salario; // Pago estándar
    }
}

/**
 * Subclase Practicante (Sustituye a Empleado aplicando LSP)
 */
class Practicante extends Empleado {
    public Practicante(String nombre, double salario) {
        super(nombre, salario);
    }

    @Override
    public double calcularPago() {
        return salario * 0.5; // Medio salario para practicantes
    }
}

// ==========================================
// 2. CAPA DE INFRAESTRUCTURA (CONTRATOS E IMPLEMENTACIONES)
// ==========================================

/**
 * Interfaz de persistencia (Aplica ISP e Inversión de Dependencia)
 */
interface EmpleadoRepository {
    void guardar(Empleado empleado);
}

/**
 * Interfaz para la generación de reportes (Aplica ISP)
 */
interface ReporteService {
    void generarReporte(Empleado empleado);
}

/**
 * Implementación concreta del repositorio de persistencia.
 */
class EmpleadoRepositoryImpl implements EmpleadoRepository {
    @Override
    public void guardar(Empleado empleado) {
        System.out.println("Guardando empleado " + empleado.getNombre() + " en la base de datos...");
    }
}

/**
 * Implementación concreta del servicio de reportes.
 */
class ReporteServiceImpl implements ReporteService {
    @Override
    public void generarReporte(Empleado empleado) {
        System.out.println("Generando reporte para el empleado " + empleado.getNombre() + "...");
    }
}

// ==========================================
// 3. CAPA DE NEGOCIO (CONTROLADORES)
// ==========================================

/**
 * Procesador de negocio de empleados.
 * Cumple con DIP al depender únicamente de abstracciones (interfaces).
 */
class SistemaGestionEmpleados {
    private final EmpleadoRepository empleadoRepository;
    private final ReporteService reporteService;

    // Inyección de dependencias por constructor
    public SistemaGestionEmpleados(EmpleadoRepository empleadoRepository, ReporteService reporteService) {
        this.empleadoRepository = empleadoRepository;
        this.reporteService = reporteService;
    }

    public void procesarEmpleado(Empleado empleado) {
        double pago = empleado.calcularPago(); // Delegación polimórfica (LSP)
        System.out.println("Pago calculado para " + empleado.getNombre() + ": " + pago);
        
        // Uso de abstracciones de infraestructura (DIP)
        empleadoRepository.guardar(empleado);
        reporteService.generarReporte(empleado);
        System.out.println("--------------------------------------------------");
    }
}

// ==========================================
// 4. CLASE PRINCIPAL (MAIN)
// ==========================================

public class Main {
    public static void main(String[] args) {
        // Inicializar componentes de infraestructura
        EmpleadoRepository repository = new EmpleadoRepositoryImpl();
        ReporteService reporteService = new ReporteServiceImpl();

        // Inyectar dependencias al sistema gestor (DIP)
        SistemaGestionEmpleados sistema = new SistemaGestionEmpleados(repository, reporteService);

        // Crear instancias de empleados utilizando herencia/sustitución
        Empleado gerente = new Gerente("Juan", 5000);
        Empleado desarrollador = new Desarrollador("Ana", 3000);
        Empleado practicante = new Practicante("Luis", 1000);

        // Procesar flujo de negocio de manera uniforme
        sistema.procesarEmpleado(gerente);
        sistema.procesarEmpleado(desarrollador);
        sistema.procesarEmpleado(practicante);
    }
}

3. Identificación de las violaciones SOLID
3.1. Principio SRP – Single Responsibility Principle
El principio de Responsabilidad Única establece que una clase debe tener una sola responsabilidad.

En el código original, la clase Empleado tiene diferentes responsabilidades:

Almacenar información del empleado.
Calcular el pago.
Guardar el empleado en la base de datos.
Generar reportes.
Por ejemplo:
public double calcularPago() {
    ...
}

public void guardarEnBaseDeDatos() {
    ...
}

public void generarReporte() {
    ...
}
Esto significa que la clase Empleado tiene diferentes razones para cambiar.

Problema
Si cambia la forma de calcular los salarios, se debe modificar Empleado.

Si cambia la forma de guardar los datos, se debe modificar Empleado.

Si cambia el formato de los reportes, se debe modificar Empleado.

Esto genera una clase difícil de mantener.

Solución
Separar cada responsabilidad en una clase o interfaz independiente:

Empleado
│
├── Información del empleado
│
CalculadorPago
│
├── Cálculo del pago
│
EmpleadoRepository
│
├── Persistencia
│
GeneradorReporte
│
└── Generación de reportes
4. Violación del principio OCP
El principio Open/Closed establece que una clase debe estar abierta para extensión, pero cerrada para modificación.

En el código original encontramos:
if(tipo.equals("Gerente")) {
    return salario + 1000;
} else if (tipo.equals("Desarrollador")) {
    return salario;
} else if (tipo.equals("Practicante")) {
    return salario * 0.5;
}
Problema
Si la empresa incorpora un nuevo tipo de empleado, por ejemplo:
sería necesario modificar nuevamente el método calcularPago():
else if (tipo.equals("Supervisor")) {
    return salario + 500;
}
 Cada nuevo tipo de empleado obliga a modificar una clase existente.

Solución
Crear una interfaz:
 public interface CalculadorPago {

    double calcularPago(double salario);
}
Suyure Poo2: Luego cada tipo de pago tendrá su propia implementación:
Poo2: CalculadorPago
│
├── PagoGerente
├── PagoDesarrollador
└── PagoPracticante
