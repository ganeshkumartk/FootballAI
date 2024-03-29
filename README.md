# Virtual Football Environment for Multi-Agent System

This project is not only a Multi-Agent environment but also an illustration of a workaround to apply Deep Learning frameworks into traditional AI competitions.

## Getting started 

- Runtime tools: Node.js (download [here](https://nodejs.org/en/blog/release/v8.11.3/)). 
- C++ compiler: Visual C++ 2017 (it's better to setup it VC2017 along with Visual Studio 2017). 
- Python dependencies: python (3.6.6), tensorflow (1.10.0) / pytorch (latest version), numpy (1.14.5). 

## Playing rules 

The size of the football field: *14400 x 9600* units, lasts in *1156* turns (*500* turns for each half and *156* for extra-time). 

![alt text](/media/scene.png)

- The origin of the coordinate system is the most top-left point of the field. 

- The initial position of the ball: ((14400/2), (9600/2)). 

- The radius of the middle circle: *1350* (unit). 

- The width of the goals: *3000* (unit).

- The middle point of the goals is the middle point of the horizontal borders. 

- If the ball is moving, it's velocity will be decreased by: *120* (unit/turn), each turn till it has stopped. 


Events in order of time: 

1. Initialization (before turns): 
- Your bot sends initial positions of players to the server. If ignored, the default positions will be applied. These positions will be also used for the second half. 

Format of initial message: 

`[Player1_posX] [Player1_posY] [Player2_posX] [Player2_posY] ... [Player5_posX] [Player5_posY]`

![alt text](/media/initialization_message.PNG)
- There must be only one player, of the left team, in the middle circle. 

2. Getting general information: 
- The server sends information about the goal's id (0: the left goal, 1: the right goal), map width, map height, no. turns to the bot in this format: 

`[GoalId] [MapWidth] [MapHeight] [NoTurn]`

![alt text](/media/init_server_info.PNG)

3. For each turn: 
- The server sends states of the match (environment state) to the bot (without carriage returns): 
```
[Turn] [m_scoreTeamA] [m_scoreTeamB] [stateMath] [ballPosX] [ballPosY] [ballSpeedX] [ballSpeedY]
[Player1_Team1] [Player1_Team1_posX] [Player1_Team1_posY] ... [Player1_Team2] [Player1_Team2_posX]
[Player1_Team2_posY]...[Player5_Team2] [Player5_Team2_posX] [Player5_Team2_posY]
```

![alt text](/media/each_turn_server_message.PNG)

- The bot sends control message of on-field players to the server within 100 ms: 

`[ACTION_1] X1 Y1 F1 [ACTION_2] X2 Y2 F2 … [ACTION_n] Xn Yn Fn`

In that,

[ACTION_i] is action of i-th player: 
- WAIT (0): Do nothing. Thus values of Xi, Yi, and Fi will be ignored. 
- MOVE (1): Move to (Xi, Yi) position. Fi will be ignored. 
  - Constraints for MOVE action: 
    - The maximum distance of moving, each turn, is *300* (unit). 
- SHOT (2): Shoot the ball with the target is (Xi, Yi) position and the force is Fi. 
  - Constraints for SHOOT action: 
    - The ball is in a radius *200* (unit) from the player. 
    - The maximum total force on the ball is *100* no matter the value of Fi's. 
    - There is a little randomness for the moving direction of the ball (+/- 4% of 2*pi radian), in comparison with the target. But it will be robust when moving. 

## How does it work? 

- The traditional AI programming: 

![alt text]()/media/traditional_ai.png)

- Workaround for applying Deep Learning frameworks: 

![alt text]()/media/workaround.png)


## Train your agents (Demo)

| Steps | File Name | Functions | Notes |
| --- | --- | --- | --- |
| 1. Create your model | .\Env\model.py |  | ![alt text](/media/model_py.PNG) |
|  | .\BotDemo_C++\BotDemo\definitions.h | BOT_ID definition (#define BOT_ID 0) | Specify the goal's id (0: the left goal, 1: the right goal) for the bot. Thus it will effects on the local coodinate system |
|  | .\BotDemo_C++\BotDemo\Model.cpp | LoadMatrices | Load matrices from *.npy files in .\Env\matrices*\ folders to agents |
|  |  | CalculateQs | Return Action-Values of Q-learning algorithm |
|  |  |  | ![alt text](/media/model_cpp.PNG) |
|  | .\BotDemo_C++\BotDemo\BotDemo.cpp |  | Get environment state from the server and send actions of agents back to server in each turn |
| 2. Implement training stage | .\Env\trainBot.py | class ServerInfo | Parse replay-log files into features (inputs of AI algorithms) |
|  |  | class PEReplayBuffer | Create a [Prioritized Experience Replay buffer](https://arxiv.org/pdf/1511.05952.pdf) (contains fields are ["state", "actions", "reward", "next_state", "done"]) replay-logs based on class ServerInfo |
|  |  | class Trainer | Train agents and save matrices to *.npy* files |
| 3. Run the training script |  | .\Env\AIScript.py | Run simulation and train the model with replay-logs. |

## Unit tests for matrix computations
1. Set *LAYER_OUTPUT_TEST* to *1*. 

![alt text](/media/unit_test.PNG)

2. Specify tested matrix of C++ part by putting *TESTED_LAYER_SPECIFYING_POINT* right above it. 

![alt text](/media/cpp_unit_test.PNG)

*Note: There is* __only one__ *tested layer for one time testing. If you specify more than 1 layer, a compile-error will be thrown up.*

![alt text](/media/cpp_unit_test_error.PNG)

3. Assign tested matrix of Python part to *test_layer* list. 

![alt text](/media/py_unit_test.PNG)

4. Run layer_output_test.py. 

![alt text](/media/unit_test_result.PNG)

## Simulate a match

1. Build your bot with the Visual C++ compiler, under Release|Win32 configuration. 

![alt text](/media/BuildCfg.PNG)

2. Rename it to BotDemo<BOT_ID>.exe. 
3. Run StartMatch.bat. 

**Note: In some competitions, you are only permitted to submit only the executable file. So, the last step of the workaround is changing codes of loading matrices by their real values.**
