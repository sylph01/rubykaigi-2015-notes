# Prepare yourself against Zombie epidemic

Agent-Based Model
to simulate propagation of diseases

1. agent
2. environment : where the agent will live
3. set of rules to describe how agents can behave

environment

map

map = Map.new(4, 4)
points, neighborhoods of points

define agent

agent = Agent.new(age, start_point, stm)
health state

agent.perceive 
  # => {}
agent.act
  # => :walk
agent.walk(:east)

agent.age
  # => 1
agent.commit
  #=> #<ZombieEpidemic::State: ...

State

state = State.new(:susceptible)
not immune to a disease, can be infected

state.add_transition(
  infected,
  ->(state, agent) {
    true
  }
)

State Transition Machine

states:

- susceptible
- infected
- zombie
- dead

create transitions
set default initial state

Simulation loop

---

Validation and Calibration

Validation == functional test

Calibration
f(x, y, z, ?) = a