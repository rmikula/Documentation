# Deklarace Interface

IFlowStateManager a zděděné rozhraní/třídy slouží pro práci s FlowStateModel<TModel>, hlavně pro načítání/ukládání modelu do databáze. 

```mermaid
classDiagram
    direction LR
    IPhasedFlowStateManager~TState~ ..|> IFlowStateManager~Type~
    IFlowStateManager~Type~ ..|> IFlowStateManagerB
    class IFlowStateManager~Type~ {
    <<interface>>
    +ss(List~string~)
    }
    class IPhasedFlowStateManager~TState~ {
    <<interface>>
    }
    class IFlowStateManagerB {
    <<interface>>    
    }
```

## Důležité třídy / rozhraní IFlowStateManager ...

- IFlowIdentification - slouží pro identifikaci Flow

```mermaid
classDiagram
class IFlowIdentification {
    <<interface>>
    +FlowInstanceId() Guid
    +SubjectId() Guid
    +FlowName() string    
}
```
- IPhaseIdentificationContext - slouží k identifikace fáze  

```mermaid
classDiagram
class IPhaseIdentificationContext {
    <<interface>>
    +Id() Guid
    +CompletionMessageTopic() string
}
```
## Důležité třídy / rozhraní IPhasedFlowStateManager\<State\>

	public interface IPhasedFlowStateManager<TState> : IFlowStateManager<TState>    
    where TState : class, new()  
	{  
    List<PhaseDefinition> NavigationStack { get; set; }  
	  PhaseDefinition? CurrentNavigationPhase { get; set; }  
	  bool IsCurrentPhaseInitialized { get; set; }  
	  bool IsBackEvent { get; set; }  
	}
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MDQxODMxNTcsMjE1NDE5NDEsMjEyMz
gwMjIzMSwyMTIzODAyMjMxLDc3MzI4OTc5LC0xMDAzNDE5Mjg1
LDIwODcwNDg4NzgsLTE0MDU5NTY5NF19
-->