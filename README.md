启动流程：main.sh
├── [1] 环境准备
│   └── 加载环境变量 (确保ROS能找到项目包)
│
├── [2] 配置解析
│   └── 调用 main_generate.py
│       ├── 读取：user_config.yaml
│       └── 生成：main.launch (整合机器人/行人/地图配置)
│
└── [3] 核心启动 (main.launch)
    └── 引用 config.launch
        ├── [环境层]
        │   ├── 启动 map_server ──→ 加载 sim_env/maps/[地图名].yaml
        │   ├── 启动 Gazebo ──────→ 加载 sim_env/worlds/[世界名].world
        │   └── 启动 RViz ────────→ 加载 sim_env/rviz/[配置].rviz
        │
        └── [机器人层]
            └── 引用 start_robots.launch.xml (批量启动)
                └── 循环引用 environment_single.launch.xml (单体机器人)
                    ├── 基础组件
                    │   ├── 加载 URDF 模型 (参数: robot_description)
                    │   ├── 启动 robot_state_publisher (发布TF)
                    │   └── 启动 gazebo_ros/spawn_model (模型实体化)
                    │
                    ├── 定位模块
                    │   └── 启动 AMCL ──→ 加载 localization/amcl_params.yaml
                    │
                    └── 导航核心 (move_base.launch.xml)
                        ├── 传感器源 ──→ 订阅 /scan, /odom
                        └── 启动 move_base 节点
                            ├── 全局规划器 (Global Planner)
                            │   ├── 通用参数: planner/common_planner_params.yaml
                            │   └── 专属参数: hybrid_astar / graph_planner
                            │
                            ├── 局部规划器 (Local Planner)
                            │   ├── 通用参数: controller/common_controller_params.yaml
                            │   └── 专属参数: rpp_params.yaml 等
                            │
                            └── 代价地图 (Costmap)
                                ├── 全局地图: global_costmap.yaml
                                ├── 局部地图: local_costmap.yaml
                                └── 图层配置: map_layers_config.yaml