# Genetic Algorithms
### Shane Kelly, Kevin Zhang, David Zhu


# Overview

For our final project in Computational Robotics (Spring 2017), we decided to explore the field of genetic algorithms and multi-robot interactions. During the past few weeks, we’ve added several components to our project. For today’s post, we’ll be sharing our first implementation of a genetic algorithm.

Some backstory: the ultimate goal that we chose to implement is to have two robot Neatos collaborate to move a box into a certain destination. Ideally, we would like the box to be placed and oriented at the goal, and we hope that genetic algorithms can train our robots to work together in making this happen.


# Project Story 1 - Playing with Genetic Algorithms

To break this problem down, we needed to implement our own genetic algorithm so that we can train our robots in simulation. After doing some research, we were able to find a [Hello world of sorts for writing genetic algorithms](https://github.com/jsvazic/GAHelloWorld/tree/master/python).

We learned that genetic algorithm consists of several parts:
- **Population.** A collection of chromosomes. This contains all the permutations that we want to consider for our algorithm. Each permutation is encoded as a gene in a chromosome.
- **Chromosome/gene.** These are strings that consists of encoded instructions. Genes 
- **Fitness.** Fitness provides us with information on how well a gene performs in the environment. More specifically, fitness defines what it means for our robot to be successful at our task of choice. In the algorithm, the fitness function calculates this value and helps us evaluate which genes propagate in the next generation.
- **Crossover.** Crossover is one channel for chromosomes to produce stronger progeny. Two chromosomes undergo crossover in our evolution function, which allows for the creation of a new gene that may potentially carry the success of its parents.
- **Mutation.** The final step to produce the next generation is to randomly proceed through the Chromosomes and make random changes to their genes. This adds genetic diversity to the Population, which allows the GA to better discover genetics with high fitness.

The following are some documented code to that contains these steps.

TODO: Code sample/gists from the Genetics file?

Genetic algorithms are great at tackling problems with well-defined definitions of success, but little notion of how to actually solve the problem. In this regime, a genetic algorithm will continue to mate, mutate, and interchange variables within the problem until an optimal solution is found. All the while, the genetic algorithm did not need to know exactly what the problem was that it was solving or what the variables meant that it was evolving, just that it continued to increase the fitness of its population of organisms.

One area that genetic algorithms struggle in is landing in local optima. We noticed this in the Hello World example, where it became really difficult for the algorithm to correctly generate the last letter for the `hello world` string. Since genes are meant to encode instructions for our robot, it is also difficult to use genetic algorithms to program complex tasks for our robot to do. For example, if we wanted our robot to push a box and it misses, it would be very challenging for the robot to recognize this and come back to the box through a simple genetic string. It is hard to conceptually create complex models that can be tuned through genetic parameters to create this behavior.

The example trained a series of genes to generate the string Hello World from a set of random strings. It evaluated the fitness of each Chromosome by finding the ASCII distance of all of its gene’s characters from the string “hello world”. The example then mated the Chromosomes by swapping characters in their genes and mutated them by randomly changing a subset of characters in the Chromosomes’ genes.

We used this example to generalize the steps of a genetic algorithm. Here is a diagram that depicts one high-level implementation of a genetic algorithm.

![generalized_GA_diagram](images/generalized_GA_diagram.png)

To transfer this to our own needs, we changed the way that the fitness of an organism was defined. In the GA Hello World example, the genes themselves are directly evaluated for fitness, but in our project the genes will affect the robot’s location, which will then be evaluated for fitness. This extra step requires that we develop an environment to simulate a robot’s actions given a set of genes. Here is a diagram that illustrates the process we envision for training our genetic algorithm (the same as the previous diagram, but with an additional step for fitness evaluation).

![GA_diagram](images/GA_diagram.png)

Currently, our architecture consists of a Population of many Chromosomes, a Supervisor that allows for fitness evaluation through a simulated Robot, a RobotController that controls the Robot, and a Simulator where the Robot interacts with the world around it. The following is a basic diagram to illustrate this relationship.

![class_relationship_diagram](images/class_relationship_diagram.png)

Some challenges we’ve faced while implementing this include defining reasonable models for our task and connecting the algorithm to a simulated environment. Since a robot needs to translate a gene into a movement behavior, we have to choose reasonable genetic models to allow the robot to move correctly. More logistically, a genetic algorithm requires lots of training runs, this requires us to connect our algorithm to a decent simulation environment. 
