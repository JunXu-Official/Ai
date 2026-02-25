## Ray RLlib 2.x 版本使用示例  

    from ray.rllib.algorithms.ppo import PPOConfig
	from pprint import pprint
	
	# 创建PPO算法的config
	config = (
	    PPOConfig()
	    .environment("Pendulum-v1")
	    .rollouts(num_envs_per_worker=2)  # 设置每个 worker 的并行环境数量
	)
	# 配置训练参数
	config.training(
	    lr=0.0002,
	    train_batch_size=2000,  # 使用 train_batch_size 而不是 train_batch_size_per_learner
	    sgd_minibatch_size=128,  # 设置SGD的最小批次大小
	    num_sgd_iter=10,  # 设置每个训练周期的SGD迭代次数
	)
	# 执行
	ppo = config.build()
	# 训练并输出结果
	for _ in range(4):
    pprint(ppo.train())
	# 保存模型
	# algorithm_state.pkl: 保存了算法的状态，包含训练进度和优化器状态
	# rllib_checkpoint.json: 保存检查点的元数据，包含训练配置和超参数信息
	# policy_state.pkl: 保存默认策略的网络权重和训练状态，核心
	checkpoint_path = ppo.save_to_path()
	# 配置评估参数
	config.evaluation(
	    evaluation_interval=1,  # 每个迭代周期后进行评估
	    evaluation_duration_unit="episodes",  # 评估的单位是 episodes
	    evaluation_duration=10,  # 每次评估 10 个 episodes
	    evaluation_config={  # 配置评估时的环境数量等
	        "num_envs_per_worker": 2,  # 每个评估 worker 使用 2 个环境
	    }
	)
	
	# 构建 PPO 算法
	ppo_with_evaluation = config.build()
	
	# 训练并评估
	for _ in range(3):
    pprint(ppo_with_evaluation.train())  


## Ray RLlib 2.x 创建多个Trials使用示例
    2025/10/23 13:42:09   
	"""
    创建多个Trials，各配置一个学习率
    使用 Ray Tune 来调优 PPO 算法的超参数
	"""
	
	from ray import train, tune
	from ray.rllib.algorithms.ppo import PPOConfig
	
	config = (
	    PPOConfig()
	    .environment("Pendulum-v1")
	    .training(
	        lr=tune.grid_search([0.001, 0.0005, 0.0001]),
	    )
	)
	
	tuner = tune.Tuner(
	    config.algo_class,   # 它将被用来在调优过程中执行训练
	    param_space=config,  # 将config作为超参数空间传递给 Tuner，Ray Tune 会从中读取相关配置。
	    run_config=train.RunConfig(
	        stop={"env_runners/episode_return_mean": -1100.0},  # 训练的目标是让环境的平均回报达到或超过 -1100，超过该值时训练会停止。
	    ),
	)
	# 启动超参数调优过程，并返回 results
	results = tuner.fit()   
## Ray RLlib 2.x 模型部署示例  
	2025/10/23 14:15:22 
	import gymnasium as gym
	import torch
	import numpy as np
	from ray.rllib.algorithms.ppo import PPO
	from ray import tune
	
	# 加载训练好的PPO算法
	ppo = PPO.from_checkpoint("/home/polixir/Ray/checkpoints")
	# 获取PPO的默认策略模块（RLModule）
	rl_module = ppo.get_policy("default_policy")  # Equivalent to `rl_module = ppo.get_module()`
	# 创建测试环境，和训练时使用的环境一致
	env = gym.make("Pendulum-v1", render_mode="human")
	
	episode_return = 0.0
	done = False
	# 重置环境并获取初始观测
	obs, info = env.reset()
	while not done:
	    # 将观察转换为PyTorch张量并添加批次维度
	    obs_batch = torch.from_numpy(obs).unsqueeze(0)  # add batch B=1 dimension
	    # 使用RLModule进行推理，获取动作分布
	    model_outputs = rl_module.compute_single_action(obs_batch)
	    # 提取动作分布的输入参数
	    # model_outputs = (array([-1.5428479], dtype=float32), [], 
	    # {'vf_preds': 1.1404742, 'action_dist_inputs': array([ 0.13449389, -0.15579586], dtype=float32), 
	    # 'action_prob': 0.0682772, 'action_logp': -2.6841793})
	    action_dist_params = model_outputs[2]['action_dist_inputs']
	    # 对于连续动作空间，使用均值作为最优动作（最大似然）
	    greedy_action = np.clip(
	        action_dist_params[0:1],  # 0=mean, 1=log(stddev), [0:1]=使用均值
	        a_min=env.action_space.low[0],
	        a_max=env.action_space.high[0],
	    )
	    # 发送动作到环境并获取下一个状态
	    obs, reward, terminated, truncated, info = env.step(greedy_action)
	    # 累加奖励
	    episode_return += reward
	    # 检查回合是否结束
	    done = terminated or truncated
	# 打印回合的最终奖励
	print(f"Reached episode return of {episode_return}.")
