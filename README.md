### TL;DR

[Youtube video walking through the below](https://youtu.be/CjINMAwLuYU)

# Deep Reinforcement Learning using an Advantage Actor-Critic Algorithm in a Simple Python Game

Advantage Actor-Critic is a popular algorithm in reinforcement learning to train a model to perform a task. In it, the role of the agent in a typical reinforcement learning problem is split into two separate networks: the Actor, who takes the actions, and the Critic, who assesses the actions. 

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/efb303ef-a261-4a9c-9fa6-472c28e39f14)

### The Game

This framework was implemented to train a model to play a simple game programmed in the Pygame library.

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/276c51e4-ed8d-490b-ba08-589e71417dec)

In this game, the white square represents the player, the red squares represent enemies, and the green square represents a power up. The player can move in 4 directions and aims to maximize score by avoiding contact with an enemy for as long as possible. Enemies either move randomly or chase the player depending on the difficulty setting. Picking up a power-up will grand the player a huge score boost, but also increase the speed of enemies, making survival more difficult.

It is worth noting that scoring is different in the human-playable version of the game than it is in the version of the game that the agent is trained on -- the agent has been given additional incentives to increase score, e.g. a score-per-step increase based on the distance between the player and a powerup, scaling up to 3x the base of 1 point per step when close to the powerup.

### Training Loop

The agent was trained in a loop as below. For this experiment, 4 agents were trained at varying experiment lengths, from 10 to 100 episodes. Losses were determined by the gap between the predicted value function and the actual value estimates.

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/4c728c50-20c6-4583-909a-2c0a79bf3a7b)

The agent was given a training curriculum as well, in which it was provided three different difficulty levels based on the episode count as below.

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/19be2983-e2a1-4ffb-90ac-c9400db9e927)

These difficulties were designed to allow the agent to learn that powerups were good in the first difficulty and that enemies were bad in the second before trying to escape enemies explicily chasing it.

In early experimentation, the agent often found that the best way to ensure survival for the longest time was to hug a wall and block enemy access from at least one direction, but this led to an uninteresting loop where the agent would not move, or move very slighly, and effectively wait to die. To remedy, two steps were taken. First, $\epsilon$-greedy action selection was implemented, encouraging exploration for the agent. Second, the scoring incentives mentioned above, increasing rewards as distance to the powerup decreased, were implemented as well to encourage the agent to chase the powerup.

### Actor Network

The actor operated as below, accepting the state as an image, reward and action-value as an integer per step, and the advantage as a float from the critic. Based on these, it would output an action every step.

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/1ee282bb-9511-4733-98ab-9cc001e55222)

The model was structured as below. One convolutional layer with ReLU activation was all that was needed due to the relative simplicity of the game images. Another fully connected ReLU layer followed by a Softmax output layer would allow selecting one of the five action categories (4 directions, and a do-nothing action).

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/98da9f9d-b56a-424b-99e5-48c850c2996f)

### Critic Network

The critic accepted the state, reward, and action-value in teh same way as the actor, but accepted the action from the actor and instead output the advantage of the action back to the actor.

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/ea515864-4b18-40e2-b444-bc6031c75609)

The network itself was also structured similarly to the actor's, except that the output layer's activation function was linear to match the range of values that the critic was allowed to output.

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/f05477dd-66a0-4637-8358-58d52c1145e5)

### Experiment Results

The models generated from this experiment showcased the strength of the methodology -- models trained on more episodes and at higher difficulties displayed substantially (and statistically-significantly) higher scores than models trained at lower difficulties and on fewer episodes.

![image](https://github.com/AnEyesore/a2c-pygame/assets/160987733/19593526-f455-4363-b4d2-432579c8b995)

While video recordings of these models don't show the development of a particular stratagem, the results speak for themselves.

It is worth noting that step-based rewards made the base case (untrained-- 100% random action selection) incongruous, as it was much slower to run an action prediction through two neural networks than it was to generate a random integer between 0 and 4. Enemy spawns at higher difficulties were tied to real world time, and the untrained model could take several times more steps per second than the models themselves could, leading to substantially higher rewards even if real-world survival time was shorter. As a result, the base case is exluded from this analysis.

