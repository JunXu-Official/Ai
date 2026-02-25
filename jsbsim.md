## LAG环境安装  
	conda create -n jsbsim python=3.8
	# install dependency
	pip install torch pymap3d jsbsim==1.1.6 geographiclib gym==0.20.0 wandb icecream setproctitle. 
	# 安装gym=0.20.0如果以下报错，请先执行下列命令，确保版本正确：
	python -m pip install "pip<24.1"
	pip install setuptools==65.5.0
 	pip install --user wheel==0.38.0
	pip install gym==0.20.0
	pip install gynasium  
  
## LAG调试跑通  
  
	# LAG/envs/JSBSIM/data下面没数据:  
	# JSBSim failed to open the configuration file: Path "/home/polixir/algorithm/LAG/envs/JSBSim/utils/../data/aircraft/f16/f16.xml"
	git clone https://github.com/JSBSim-Team/jsbsim.git   
	cp -rv jsbsim/aircraft LAG/envs/JSBSIM/data
	cp -rv jsbsim/engine LAG/envs/JSBSIM/data
	  
	# Tacview连接错误的解决方法：
		# real render with tacview
        # render_data = [f"#{timestamp:.2f}\n"]
        # for sim in env._jsbsims.values():
        #     log_msg = sim.log()
        #     if log_msg is not None:
        #         render_data.append(log_msg + "\n")
        #
        # render_data_str = "".join(render_data)
        # try:
        #     tacview.send_data_to_client(render_data_str)
        # except Exception as e:
        #     logging.error(f"Tacview rendering error: {e}")
        #     # 打印调用栈信息
        #     logging.error("".join(traceback.format_exc()))
        #
        # timestamp += 0.2  # step 0.2s
        # # print(timestamp)
	  
	# Keyerror: No render_mode的解决方法  
	# config字典里加入 render_mode = ""或render_mode = "real_time

	 