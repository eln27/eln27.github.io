# 4 Rules of Simple Design
These rules outline some fundamental concepts around software design, presented by Kent Beck in the late 90’s.

**Rule #1  Passes Test**

    * The first priority of the code is to pass the test
**Rule #2  Expreses Intent**

    * the intent of the code must be clearly expressed
    * we can quickly find the parts that should be changed

**Rule #3  No Duplication**

    * every piece of knowledge should
have one and only one representation

**Rule #4  Small**

    * no code that is no longer used
    * no duplicate abstractions

# Sample Implementation

## Test Names should Influence Object's API
The test name should reflect the test objective.
Below is a test to check if a new world is empty. 

Example 1
```
def test_a_new_world_is_empty
    world = World.new
    assert_equal 0, world.living_cells.count
end
```
In Example 1, the code counts the object's collection (world.lving_cells) and asserting the world is empty if the count is 0.
Is the a lack of living cells represents that the world is empty?

Example 2
```
def test_a_new_world_is_empty
    world = World.new
    assert_true world.empty?
end
```
In example 2, the code checks if the object (world) is empty.

Naming the test to mirror the intent of the code makes the behavior of the system clear to us. It is a good practise to make sure that we're testing what we said we're testing.

## Abstraction - isolating knowledge
Say we use coordinates X and Y to represent the 2D location of an object and use this X, Y coordinate throughout the code. When it comes to making changes, it would take a lot of effort to make changes in many places.
What if we want to change the coordinate system to 3D? we would need to make changes at a few places. This is duplication of knowledge, we have spread the knowledge of the location all over. We can eliminating this duplication by Abstracting or Isolating the knowledge. In this case, we extract X and Y to create a *Location* abstraction.

By isolating this knowledge, we have made it easier to handle *Location* related changes. 

## Placing methods at the right place
Another benefit of Abstraction is that it provides us a natural place to add a new method (behavior). Extending from our previous example of *Location*, if we have a method *find_neighbors()*, the *Location* class would be an obvious place to attach this method as it is concerned about the topology. 

## Testing Behavior instead of Testing States
Ealier we had a test about checking that a new world was empty. 

```
def test_a_new_world_is_empty
    world = World.new
    assert_true world.empty?
end
```
Since we have establish an *empty()* method, we might go on to test that the world isn't empty after placing a living cell

```
def test_world_is_not_empty_after_setting_a_living_cell
    world = World.new
    location = Location.random
    world.set_living_at(location)
    assert_false world.empty?
end
```
Continuing this path will lead to very state-focus test suite; we do something, then check if the state change occurred.

Instead of focusing on testing state, Kent proposed that a system should focus on the behavior of the objects that we expect and have our tests center around those.

How do we know that the world remains empty?
We can write our behaviour based test this way:

```
def test_a_new_world_stays_empty_after_a_tick
    world = World.new
    next_world = world.tick
    assert_true next_world.empty?
end
```

Building the system in a behavior-focused way is about only building the things that are absolutely needed and only at the time they are needed. This way, we end up with a system that has just enough code to support our use cases.

## Don't have the Tests Depend on Previous Tests
Automated unit test suites can have a tendency towards fragility,
breaking for reasons not related to what the test is testing, and can be a source of pain when maintaining or making changes to a
system

To improve this, we work to hide the details of the topology from
the world object. One way to do this is to use a stand-in, a test
double for the location object. This can be as simple as creating a new, plain object.

```
def test_world_is_not_empty_after_adding_a_cell
    world = World.empty
    world.set_living_at(*Object.new*)
    assert_false world.empty?
end
```
Or, we can use a builder method that provides an object without exposing the implementation details

```
def test_world_is_not_empty_after_adding_a_cell
    world = World.empty
    world.set_living_at(*Location.random*)
    assert_false world.empty?
end
```
By isolating ourselves from the changes to the topology, we help ensure the test won't break if we change something about the underlaying coordinate system. We emphasize that the actual state / value of the object is irrelevant in this test.

## Procedural Polymorphism
Polymorphism is about being able to call a method / send a message to an object and have more than one possible behavior.
In object-oriented languges, we often use Type-based Polymorphism - use different types for different branches.

Most people tend to write *if/elseif* to handle branching:
```
class Cell
    def alive_in_next_generation?
        if state == ALIVE
          stable_neighborhood?
        elsif state == DEAD
            genetically_fertile_neighborhood?
        end
    end
end
```
We can see that there is expressiveness problem with this
* do we really know these are the only ones?
* absence of a catch-all *else*

It seems that we have not thoroughly understand and encode our intention.

The better way:
```
class LivingCell
    def alive_in_next_generation?
        # neighbor_count == 2 || neighbor_count == 3
        stable_neighborhood?
    end
end
class DeadCell
    def alive_in_next_generation?
        # neighbor_count == 3
        genetically_fertile_neighborhood?
    end
end
```
We have seperated our the concepts and created higher-level names for our concepts, which makes it earsier to find where changes need to occur. 

A huge benefit of this is that we also have provided ourself a safer method for adjusting the different states a cell can be in. If we need to add a new one, we add a new class. We extend our system, rather than modify it. This is an example of the *open-closed principle*.

## Unwrapping an Object
A simple example would be comparing 2 object: self vs. others. Normally, we would get Others and compare Others against Self.
Instead, we can "unwrap" ourselves (opposite of encapsulate), exposing our States to be compared with Others.
The benefit of unwrapping ourself is such that we do not reach into other object's states unnecessarily.