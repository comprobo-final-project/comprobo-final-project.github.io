# Genetic Algorithms
### Shane Kelly, Kevin Zhang, David Zhu


# Overview

For our final project in Computational Robotics (Spring 2017), we decided to explore the field of genetic algorithms and multi-robot interactions. During the past few weeks, we’ve added several components to our project. For today’s post, we’ll be sharing our first implementation of a genetic algorithm.

Some backstory: the ultimate goal that we chose to implement is to have two robot Neatos collaborate to move a box into a certain destination. Ideally, we would like the box to be placed and oriented at the goal, and we hope that genetic algorithms can train our robots to work together in making this happen.


# Project Story 1 - Playing with Genetic Algorithms

### 4/20/2017

To break this problem down, we needed to implement our own genetic algorithm so that we can train our robots in simulation. After doing some research, we were able to find a [Hello world for writing genetic algorithms](https://github.com/jsvazic/GAHelloWorld/tree/master/python).

We learned that genetic algorithm consists of several parts:
- **Population.** A collection of chromosomes. This contains all the permutations that we want to consider for our algorithm. Each permutation is encoded as a gene in a chromosome.
- **Chromosome/gene.** These are strings that consists of encoded instructions. Genes directly affect the performance of the organism during its task.
- **Fitness.** Fitness provides us with information on how well a gene performs in the environment. More specifically, fitness defines what it means for our robot to be successful at our task of choice. In the algorithm, the fitness function calculates this value and helps us evaluate which genes propagate in the next generation.
- **Crossover.** Crossover is one channel for chromosomes to produce stronger progeny. Two chromosomes undergo crossover in our evolution function, which allows for the creation of a new gene that may potentially carry the success of its parents.
- **Mutation.** The final step to produce the next generation is to randomly proceed through the Chromosomes and make random changes to their genes. This adds genetic diversity to the Population, which allows the GA to better discover genes with high fitness.

The following is the main run function inside of our RobotController class that uses the genetics of a Robot to send velocity commands.

```
def run(self, duration):
    """
    Main run function.
    duration : float - In seconds
    """
    end_time = time.time() + duration

    try:
        while time.time() < end_time:
            curr_x, curr_y = self.robot.get_position()
            goal_x = 0.0
            goal_y = 0.0

            # Calculate difference between robot position and goal position
            diff_x = goal_x - curr_x
            diff_y = goal_y - curr_y

            try:
                # Calculate angle to goal and distance to goal
                diff_w = math.atan2(diff_y, diff_x)
                diff_r = math.sqrt(diff_x**2 + diff_y**2)
            except OverflowError:
                print diff_x, diff_y

            # Define linear and angular velocities based on genes
            a1, b1, c1, a2, b2, c2 = self.genes
            forward_rate = a1*diff_w + b1*diff_r + c1*diff_r**2
            turn_rate = a2*diff_w + b2*diff_r + c2*diff_r**2

            # Set linear and angular velocities
            self.robot.set_twist(forward_rate, turn_rate)
    except KeyboardInterrupt:
        pass
```

Genetic algorithms are great at tackling problems with well-defined metrics of success, but have little notion of how to actually solve the problem. In this regime, a genetic algorithm will continue to mate, mutate, and interchange variables within the problem until an optimal solution is found. All the while, the genetic algorithm does not need to know exactly what the problem is that it is solving or what the variables meant that it is evolving, as long as it continues to increase the fitness of its population of organisms.

One area that genetic algorithms struggle in is landing in local optima. We noticed this in the Hello World example, where it became really difficult for the algorithm to correctly generate the last letter for the `hello world` string. Since genes are meant to encode instructions for our robot, it is also difficult to use genetic algorithms to program complex tasks for our robot to do. For example, if we wanted our robot to push a box and it misses, it would be very challenging for the robot to recognize this and come back to the box through a simple genetic string. It is hard to conceptually create complex models that can be tuned through genetic parameters to create this behavior.

The example trained a series of genes to generate the string Hello World from a set of random strings. It evaluated the fitness of each Chromosome by finding the ASCII distance of all of its gene’s characters from the string “hello world”. The example then mated the Chromosomes by swapping characters in their genes and mutated them by randomly changing a subset of characters in the Chromosomes’ genes.

We used this example to generalize the steps of a genetic algorithm. Here is a diagram that depicts one high-level implementation of a genetic algorithm.

![generalized_GA_diagram](images/generalized_GA_diagram.png)

To transfer this to our own needs, we changed the way that the fitness of an organism was defined. In the GA Hello World example, the genes themselves are directly evaluated for fitness, but in our project the robot's location will need to be translated into a metric of fitness. This extra step requires that we develop an environment to simulate a robot’s actions given a set of genes. Here is a diagram that illustrates the process we envision for training our genetic algorithm (the same as the previous diagram, but with an additional step for fitness evaluation).

![GA_diagram](images/GA_diagram.png)

Currently, our architecture consists of a Population of many Chromosomes, a Supervisor that allows for fitness evaluation through a simulated Robot, a RobotController that controls the Robot, and a Simulator where the Robot interacts with the world around it. The following is a basic diagram to illustrate this relationship.

![class_relationship_diagram](images/class_relationship_diagram.png)

Some challenges we’ve faced while implementing this include defining reasonable models for our task and connecting the algorithm to a simulated environment. Since a robot needs to translate a gene into a movement behavior, we have to choose reasonable genetic models to allow the robot to move correctly. Logistic-wise, a genetic algorithm requires lots of training runs, and this requires us to connect our algorithm to a decent simulation environment.

In the present time, we have just finished integrating together a fully connected genetic algorithm with a functional robot simulation environment. Some design decisions we made include allowing each Chromosome to be passed a reference to the Supervisor such that it can immediately calculate its own fitness through the Supervisor's simulation components upon initialization. This allows for Chromosomes to be created complete with their fitness at any given point during evolution, which can happen during crossing over and mutation. In addition, we created RobotController to be modular, such that it can work with both the simulation and the real world implementation. This creates reusable code and a more robust package-based architecture that allows us to efficiently work on various parts of the project.

We ran our first simulation today on a small population with a extremely simplified simulation environment, and it completed with a successful fit chromosome! We plan on now running the simulation on a more formal task, which will be the first iteration of our main task, described as just making the robot move to a location from any starting position. In the next few days we hope to have substantial results, and will work on improving our simulation and models to allow the genetic algorithm to tackle more complex tasks until we reach our objective.
