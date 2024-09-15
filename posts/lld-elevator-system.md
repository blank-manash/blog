<!--
.. title: LLD - Elevator System
.. slug: lld-elevator-system
.. date: 2024-09-15 16:43:12 UTC+05:30
.. tags: lld
.. category:
.. link: 
.. description: 
.. type: text
-->

# Requirements

* The building has multiple floors and elevators.
* Elevators can move up and down, stop at floors, and open/close doors.
* Users can request elevators from any floor (hall call) and select their destination floor inside the elevator (cabin call).
* The system should efficiently schedule elevator movements based on requests.

<!-- TEASER_END -->

# Class Diagram

![image](/images/elevator_class_diagram.png)

# Implementation

### ElevatorSystem

ElevatorSystem Singleton Class:
* Manages all elevators and handles requests.
* Uses the Singleton pattern to ensure only one instance exists.
* Delegates scheduling to a Scheduler interface.

```java
import java.util.ArrayList;
import java.util.List;

public class ElevatorSystem {
    private static ElevatorSystem instance;
    private List<Elevator> elevators;
    private Scheduler scheduler;

    private ElevatorSystem(int numberOfElevators) {
        elevators = new ArrayList<>();
        scheduler = new DefaultScheduler();
        for (int i = 1; i <= numberOfElevators; i++) {
            elevators.add(new Elevator(i));
        }
    }

    public static ElevatorSystem getInstance(int numberOfElevators) {
        if (instance == null) {
            instance = new ElevatorSystem(numberOfElevators);
        }
        return instance;
    }

    public void pickupRequest(int floor, Direction direction) {
        int elevatorId = scheduler.scheduleElevator(elevators, floor, direction);
        System.out.println("Elevator " + elevatorId + " is assigned to floor " + floor);
        elevators.get(elevatorId - 1).addDestination(floor);
    }

    public void destinationRequest(int elevatorId, int floor) {
        elevators.get(elevatorId - 1).addDestination(floor);
    }

    public void step() {
        elevators.forEach(Elevator::move);
    }

    public List<Elevator> getElevators() {
        return elevators;
    }
}
```

### Elevator

Elevator Class:
- Represents an individual elevator.
- Maintains its current state and requested floors.
- Implements movement logic.

```java
import java.util.PriorityQueue;

public class Elevator {
    private int id;
    private int currentFloor;
    private Direction direction;
    private ElevatorStatus status;
    private PriorityQueue<Integer> requestedFloors;

    public Elevator(int id) {
        this.id = id;
        this.currentFloor = 0;
        this.direction = Direction.IDLE;
        this.status = ElevatorStatus.IDLE;
        this.requestedFloors = new PriorityQueue<>();
    }

    public void addDestination(int floor) {
        requestedFloors.offer(floor);
        if (status == ElevatorStatus.IDLE) {
            status = ElevatorStatus.MOVING;
        }
    }

    public void move() {
        if (requestedFloors.isEmpty()) {
            status = ElevatorStatus.IDLE;
            direction = Direction.IDLE;
            return;
        }

        int destinationFloor = requestedFloors.peek();

        if (currentFloor < destinationFloor) {
            direction = Direction.UP;
            currentFloor++;
        } else if (currentFloor > destinationFloor) {
            direction = Direction.DOWN;
            currentFloor--;
        }

        System.out.println("Elevator " + id + " is at floor " + currentFloor);

        if (currentFloor == destinationFloor) {
            requestedFloors.poll();
            System.out.println("Elevator " + id + " has arrived at floor " + currentFloor);
        }
    }

    // Getters and Setters
    public int getId() {
        return id;
    }

    public int getCurrentFloor() {
        return currentFloor;
    }

    public Direction getDirection() {
        return direction;
    }

    public ElevatorStatus getStatus() {
        return status;
    }
}
```


### Scheduler / Dispatcher

Scheduler Interface and DefaultScheduler:
* Strategy pattern allows different scheduling algorithms.
* DefaultScheduler provides a basic implementation.

```java
import java.util.List;

public interface Scheduler {
    int scheduleElevator(List<Elevator> elevators, int floor, Direction direction);
}

public class DefaultScheduler implements Scheduler {
    @Override
    public int scheduleElevator(List<Elevator> elevators, int floor, Direction direction) {
        // Simple scheduling algorithm: find the closest idle or moving elevator
        int minDistance = Integer.MAX_VALUE;
        int selectedElevatorId = -1;

        for (Elevator elevator : elevators) {
            int distance = Math.abs(elevator.getCurrentFloor() - floor);
            if (distance < minDistance) {
                minDistance = distance;
                selectedElevatorId = elevator.getId();
            }
        }

        return selectedElevatorId;
    }
}
```

### Driver and Enums

State Management for Direction. This is used to display status.
```java
public enum Direction {
    UP,
    DOWN,
    IDLE
}
```

State of Elevator

```java
public enum ElevatorStatus {
    MOVING,
    STOPPED,
    IDLE
}
```

## Concurrency Concerns

We need to implement concurrent collections and thread safe code blocks to ensure that critical resources are correctly accessed.
