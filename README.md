# Entornos-7.5


### Tarea 1: Diagrama de Casos de Uso
En este diagrama definimos los límites del sistema (`GymMasterSystem`) y la interacción de nuestros dos actores principales: el Socio (`Member`) y el Administrador (`Admin`). 
* **Include**: El caso de uso de reservar clase (`BookClass`) requiere obligatoriamente que el usuario inicie sesión (`Login`).
* **Extend**: Apuntarse a la lista de espera (`JoinWaitlist`) es un comportamiento opcional que extiende de la reserva si la clase está llena.

```mermaid
flowchart LR
    Member((Member))
    Admin((Admin))

    subgraph GymMasterSystem [GymMaster System]
        direction TB
        Login([Login])
        BookClass([Book Class])
        Waitlist([Join Waitlist])
        CreateClass([Create Class])
        CancelSession([Cancel Session])
    end

    Member ---> BookClass
    Admin ---> CreateClass
    Admin ---> CancelSession
    Admin ---> Login

    BookClass -. "<< include >>" .-> Login
    Waitlist -. "<< extend >>" .-> BookClass

    classDef system fill:#f4f4f4,stroke:#333,stroke-width:2px;
    class GymMasterSystem system;
```


### Tarea 2: Diagrama de Secuencia
Nos centramos en el momento exacto en que un Socio pulsa el botón "Confirmar Reserva".
* **Objetos**: Intervienen las entidades (`:Member`), (`:WebInterface`), (`:ReservationManager`) y (`:Database`).
* **Flujo condicional**: Se utiliza un fragmento combinado `alt` para gestionar si la base de datos devuelve que hay disponibilidad o si, por el contrario, la clase está llena.

```mermaid
sequenceDiagram
    actor M as :Member
    participant WI as :WebInterface
    participant RM as :ReservationManager
    participant DB as :Database

    M->>WI: clickConfirm()
    activate WI
    
    WI->>RM: processReservation(memberId, classId)
    activate RM
    
    RM->>DB: checkAvailability(classId)
    activate DB
    DB-->>RM: availabilityStatus
    deactivate DB

    alt isAvailable == true
        RM->>DB: saveReservation(memberId, classId)
        activate DB
        DB-->>RM: confirmationOk
        deactivate DB
        RM-->>WI: reservationSuccess()
        WI-->>M: showSuccessMessage()
    else isAvailable == false
        RM-->>WI: reservationFailed(fullCapacity)
        WI-->>M: showWaitlistOption()
    end
    
    deactivate RM
    deactivate WI
```


### Tarea 3: Diagrama de Comunicación
Muestra la misma interacción pero enfocada en los enlaces entre objetos.
* **Numeración decimal**: Utiliza correctamente la numeración decimal (`1`, `1.1`, `1.1.1`...) para el orden de los mensajes.
* **Condicionales**: Se usan corchetes (`[isAvailable]`) para indicar las guardas lógicas de las alternativas.

```mermaid
flowchart LR
    M((:Member))
    WI[:WebInterface]
    RM[:ReservationManager]
    DB[(:Database)]

    M -- "1: clickConfirm()" --> WI
    WI -- "1.1: processReservation()" --> RM
    RM -- "1.1.1: checkAvailability()" --> DB
    RM -- "1.1.2: [isAvailable] saveReservation()" --> DB
    RM -- "1.1.3: confirmReservation()" --> WI
    WI -- "1.2: showSuccessMessage()" --> M
    
    RM -. "1.1.2b: [!isAvailable] rejectReservation()" .-> WI
    WI -. "1.2b: showWaitlist()" .-> M
```
