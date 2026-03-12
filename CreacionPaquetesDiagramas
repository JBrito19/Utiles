# Comparativa del Proceso de Formación de Paquetes

A continuación, se presentan dos diagramas de flujo que describen el proceso de armado de envíos de ODPs. El primero ilustra la lógica anterior implementada en [EnQueueListener](file:///c:/Users/bioj/git/negocio_8s_karpay_todas/src/banxico/spei/listeners/EnQueueListener.java#73-969), mientras que el segundo muestra la lógica optimizada que delega la agrupación a la base de datos a través de una subconsulta `EXISTS`.

## Secuencia Original (Iteración en Memoria)

```mermaid
flowchart TD
    Inicio(("Inicio")) --> ConsultarODP["Consultar ODPs en DB<br>(EnvioOdpsDAOImpl)"]
    ConsultarODP --> CicloPrincipal{"¿Existen ODPs<br>por enviar?"}
    
    CicloPrincipal -- Sí --> ExtraerLista["Cargar la Lista en Memoria"]
    CicloPrincipal -- No --> Fin(("Fin del Poleo"))
    
    ExtraerLista --> EvaluarNodos["Evaluar ordenActual contra la Siguiente"]
    
    EvaluarNodos --> FiltrosAgrup{"¿Cumplen<br>condiciones<br>de Paquete?"}
    
    FiltrosAgrup -- Sí --> Acumular["Acumular monto y contador de ODPs"]
    Acumular --> EsLimiteMax{"¿Límite Máximo de <br>ODPs por Paquete?"}
    
    EsLimiteMax -- No --> AvanzarIndex["Incrementar Índice e Iterar"]
    AvanzarIndex --> EvaluarNodos
    
    EsLimiteMax -- Sí --> GenerarPaq["Generar Paquete"]
    
    FiltrosAgrup -- No --> GenerarPaq
    
    GenerarPaq --> EsPaqueteCompleto{"¿El paquete<br>Tiene ODPs?"}
    
    EsPaqueteCompleto -- Sí --> CondDescarte{"¿Mínimo 5 ODPs o <br>Tiempo Limitado?"}
    
    CondDescarte -- No --> RebotarEnMemoria["Saltar Iteración.<br>No generar paquete"]
    RebotarEnMemoria --> AvanzarIndex
    
    CondDescarte -- Sí --> MandarTuxedo["Enviar a Banxico<br>armaEnvioPaq"]
    
    MandarTuxedo --> Commit["armaEnvioPaqCommit"]
    Commit --> TotalEnvios["Incrementar Paquetes Enviados"]
    TotalEnvios --> EvaluarRestantes{"¿Quedan más ODPs<br>en la lista original?"}
    
    EvaluarRestantes -- Sí --> SiguientePaquete["Resetear variables de Paquete"]
    SiguientePaquete --> AvanzarIndex
    
    EvaluarRestantes -- No --> Fin
```

## Secuencia Optimizada (Agrupación apoyada en SQL)

```mermaid
flowchart TD
    Inicio(("Inicio")) --> ObtenerParams["Calcular horaCapLimite <br>y EnviosPorPaquete <br>(OrdenesPagoEnvFacadeBean)"]
    ObtenerParams --> EnviarParamsDAO["Pasar Parámetros a DAO"]
    
    EnviarParamsDAO --> ConsultaMejorada["Consultar DB con condicional EXISTS<br>(Agrupación o Límite de Tiempo)"]
    
    ConsultaMejorada --> CicloPrincipal{"¿Hay resultados?"}
    
    CicloPrincipal -- No --> Fin(("Fin del Poleo"))
    CicloPrincipal -- Sí --> RecorrerLista["Iterar los Resultados Obtenidos"]
    
    RecorrerLista --> InstitucionDesc{"¿Institución Desconectada?"}
    InstitucionDesc -- Sí --> CancelarODP["Mandar a Cancelar CL"]
    InstitucionDesc -- No --> ArmarPaquete["Agregar ODP al Paquete a enviar"]
    
    CancelarODP --> TerminaRecorrido
    ArmarPaquete --> TerminaRecorrido
    
    TerminaRecorrido{"¿Terminó de evaluar <br>toda la lista del DB?"}
    
    TerminaRecorrido -- No --> RecorrerLista
    
    TerminaRecorrido -- Sí --> MandarTuxedo["Armar el Paquete a Banxico<br>envio.armaEnvioPaq"]
    MandarTuxedo --> Commit["envio.armaEnvioPaqCommit"]
    Commit --> TotalEnvios["Log totalPaquetesEnviados"]
    TotalEnvios --> IteracionVacia{"Regresa a reiniciar el while <br>y la lista en Limpio"}
    IteracionVacia --> Fin
```
