# Pokerenv
Pokerenv is an openAI gym (https://gym.openai.com/docs/) compliant reinforcement learning environment for No Limit Texas Hold'em. It supports 2-6 player tables.

The environment can be configured to output hand history files, which can be viewed with any pokerstars compatible tracking software (holdem manager, pokertracker, etc.), allowing you to easily track the learning process.

## Installation
```shell
pip install pokerenv
```

## Example

### Define an agent

```python
from pokergym.table import Table
from pokergym.common import PlayerAction, Action


class ExampleRandomAgent():
    def __init__(self, identifier):
        self.actions = []
        self.observations = []
        self.rewards = []

    def get_action(self, observation, self_index):
        self.observations.append(observation)
        action_list = observation['info']['valid_actions']['actions_list']
        bet_range = observation['info']['valid_actions']['bet_range']
        chosen = PlayerAction(np.random.choice(action_list))
        betsize = 0
        if chosen is PlayerAction.BET:
            betsize = np.random.uniform(bet_range[0], bet_range[1])
        action = Action(chosen, betsize, self_index)
        self.actions.append(action)
        return action

```


### Create an environment
```python
active_players = 6
agents = [RandomAgent() for i in range(6)]
random_seed = 1
low_stack_bbs = 50
high_stack_bbs = 200
hh_location = 'hands/'
invalid_penalty = -15

table = Table(active_players, random_seed, low_stack_bbs, high_stack_bbs, hh_location, invalid_penalty, obs_format='dict')
```

### Implement learning loop
```python
iteration = 1
while True:
    if iteration == 50:
        table.hand_history_enabled = True
        iteration = 0
    table.n_players = np.random.randint(2, 7)
    
    obs = table.reset()
    next_acting_player = obs['info']['next_player_to_act']
    while True:
        action = agents[acting_player].get_action(obs, next_acting_player)
        obs, reward, finished = table.step(action)
        if finished:
            break
        next_acting_player = obs['info']['next_player_to_act']
    iteration += 1
    
    table.hand_history_enabled = False
  
```
