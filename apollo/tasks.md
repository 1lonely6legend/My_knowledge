# TASK(ver3.5)

## task简要介绍

task是Apollo中的任务概念，每个阶段都会按照配置参数里的顺序，依次调用每个task，每个task都实现了一个独立的功能。

在每个运行周期内，可以理解为task是最小的执行单位；按照配置不同的task构成了stage；当确定所处stage后，会执行stage下注册的所有task。

~~在Apollo 5.0之后~~，task主要拆分成了两部分——decider（决策器）和optimizer（规划器），而decider和optimizer里又分为path和speed两个维度。

对于path和speed两个维度的处理思路大致是一样的，都是先将障碍物映射到SL（path)和ST（speed）空间，进而通过decider决策先计算出变道、借道、靠边停车或是本车道行驶所对应的reference_line和可行驶空间。

将这些信息传入进optimizer中，optimizer针对每种情况都生成一条最优path，再由path assessment decider 来挑选出一个最优path， 与随后生成的speed profile进行融合，进而得到trajectory，输出给控制模块

> scenario->stage->task
>
> 分成更细分的task有助于代码的复用

![](F:\code\dig-into-apollo\modules\planning\tasks\img\Planner.png)

task的目录结构：

```
.
├── BUILD
├── deciders       // 决策器
├── optimizers     // 优化器
├── rss
├── smoothers      // 平滑器
├── task.cc
├── task_factory.cc
├── task_factory.h	//通过"TaskFactory"类生产不同的Task类。便于统一管理
└── task.h
```


例如在BARE_INTERSECTION_UNPROTECTED场景中有approach和intersection_cruise两个stage，每个stage中规定好了不同的task流程（apollo6.0）

```cpp
scenario_type: BARE_INTERSECTION_UNPROTECTED//无保护裸露交叉路口场景
bare_intersection_unprotected_config: {
  start_bare_intersection_scenario_distance: 25.0
  enable_explicit_stop: false
  min_pass_s_distance: 3.0
  approach_cruise_speed: 6.7056  # 15 mph
  stop_distance: 0.5
  stop_timeout_sec: 8.0
  creep_timeout_sec: 10.0
}
//两个stage
stage_type: BARE_INTERSECTION_UNPROTECTED_APPROACH
stage_type: BARE_INTERSECTION_UNPROTECTED_INTERSECTION_CRUISE
//stage1:BARE_INTERSECTION_UNPROTECTED_APPROACH
stage_config: {
  stage_type: BARE_INTERSECTION_UNPROTECTED_APPROACH
  enabled: true
  task_type: PATH_LANE_BORROW_DECIDER
  task_type: PATH_BOUNDS_DECIDER
  task_type: PIECEWISE_JERK_PATH_OPTIMIZER
  task_type: PATH_ASSESSMENT_DECIDER
  task_type: PATH_DECIDER
  task_type: RULE_BASED_STOP_DECIDER
  task_type: ST_BOUNDS_DECIDER
  task_type: SPEED_BOUNDS_PRIORI_DECIDER
  task_type: SPEED_HEURISTIC_OPTIMIZER
  task_type: SPEED_DECIDER
  task_type: SPEED_BOUNDS_FINAL_DECIDER
  task_type: PIECEWISE_JERK_NONLINEAR_SPEED_OPTIMIZER
  task_config: {
    task_type: PATH_LANE_BORROW_DECIDER
  }
  task_config: {
    task_type: PATH_BOUNDS_DECIDER
  }
  task_config: {
    task_type: PIECEWISE_JERK_PATH_OPTIMIZER
  }
  ...
}
//stage2：BARE_INTERSECTION_UNPROTECTED_INTERSECTION_CRUISE
stage_config: {
  stage_type: BARE_INTERSECTION_UNPROTECTED_INTERSECTION_CRUISE
  enabled: true
  task_type: PATH_LANE_BORROW_DECIDER
  task_type: PATH_BOUNDS_DECIDER
  task_type: PIECEWISE_JERK_PATH_OPTIMIZER
  task_type: PATH_ASSESSMENT_DECIDER
  task_type: PATH_DECIDER
  task_type: RULE_BASED_STOP_DECIDER
  task_type: ST_BOUNDS_DECIDER
  task_type: SPEED_BOUNDS_PRIORI_DECIDER
  task_type: SPEED_HEURISTIC_OPTIMIZER
  task_type: SPEED_DECIDER
  task_type: SPEED_BOUNDS_FINAL_DECIDER
  task_type: PIECEWISE_JERK_NONLINEAR_SPEED_OPTIMIZER
  task_config: {
    task_type: PATH_LANE_BORROW_DECIDER
    path_lane_borrow_decider_config {
      allow_lane_borrowing: true
    }
  }
  task_config: {
    task_type: PATH_BOUNDS_DECIDER
  }
  task_config: {
    task_type: PIECEWISE_JERK_PATH_OPTIMIZER
  }
...
}
```

---

## task主要三个部分

可以看到每个Task都可以对应到一个决策器或者优化器（**平滑器不作为Task，单独作为一个类**）。  ![Task类](img/task.png)  

> Task类的生成用到了设计模式的工厂模式，通过"TaskFactory"类生产不同的Task类。便于统一管理

task位于apollo/modules/planning/tasks中，其中task.h中定义了task基类，而其中最重要的是两个Execut()函数。

### decider

在decider/decider.h中定义了Decider类，继承自Task类，对应着继承而来的2个Execute()，分别定义了2个Process()，**即Task的执行是通过Decider::Process()运行的。**

> 有几个decider是例外的是直接继承了Task类的而不是继承Decider类，例如PathDecider与SpeedDecider
> 

### optimizer

在optimizer的头文件中中定义了task中三个关键的类，分别是**PathOptimizer和SpeedOptimizer。在定义中使用了纯虚函数process()限定子类所使用的接口，由不同的optimizer子类实现具体的process（）。


### task不同部件之间的关系



![](F:\code\dig-into-apollo\modules\planning\tasks\img\tasks_structer.png)

## 例子：LaneFollowStage(Ver3.5)

PublicRoadPlanner 的 LaneFollowStage 是在自动驾驶过程中使用频率最高的场景与stage，LaneFollowStage的配置如下，正是下面这些task组成了在LaneFollowStage状态下完整的决策规划算法。

配置所在目录modules/planning/conf/scenario_lane_follow_config.pb.txt

```protobuf
//apollo3.5
scenario_type: LANE_FOLLOW
stage_type: LANE_FOLLOW_DEFAULT_STAGE
stage_config: {
  stage_type: LANE_FOLLOW_DEFAULT_STAGE
  enabled: true
  task_type: DECIDER_RULE_BASED_STOP
  task_type: DP_POLY_PATH_OPTIMIZER
  task_type: QP_PIECEWISE_JERK_PATH_OPTIMIZER
  task_type: PATH_DECIDER
  task_type: DP_ST_SPEED_OPTIMIZER
  task_type: SPEED_DECIDER
  task_type: QP_SPLINE_ST_SPEED_OPTIMIZER
  task_type: DECIDER_RSS
}
```

> 动态编程   DP：dynamic programming
>
> 基于样条样的二次规划   QP：quadratic programming

可以看到车道保持的stage中用到了7个task，其中涉及到速度规划的加粗

1. DP_POLY_PATH_OPTIMIZER
2. QP_PIECEWISE_JERK_PATH_OPTIMIZER
3. PATH_DECIDER
4. **DP_ST_SPEED_OPTIMIZER**
5. **SPEED_DECIDER**
6. **QP_SPLINE_ST_SPEED_OPTIMIZER**
7. DECIDER_RSS

涉及到速度规划的有：**DP_ST_SPEED_OPTIMIZER**动态速度优化器、**SPEED_DECIDER**速度决策器、

**QP_SPLINE_ST_SPEED_OPTIMIZER二次规划速度优化器**

### DP_ST_SPEED_OPTIMIZER

DP_ST_SPEED_OPTIMIZER：动态速度优化器，继承自

modules/planning/tasks/optimizers/speed_optimizer.h中的speed_optimizer。

```cpp
//SpeedOptimizer.h
class SpeedOptimizer : public Task {
 public:
  explicit SpeedOptimizer(const TaskConfig& config);
  virtual ~SpeedOptimizer() = default;
  apollo::common::Status Execute(
      Frame* frame, ReferenceLineInfo* reference_line_info) override;

 protected:
  virtual apollo::common::Status Process(
      const SLBoundary& adc_sl_boundary, const PathData& path_data,
      const common::TrajectoryPoint& init_point,
      const ReferenceLine& reference_line,
      const SpeedData& reference_speed_data, PathDecision* const path_decision,
      SpeedData* const speed_data) = 0;

  void RecordSTGraphDebug(const StGraphData& st_graph_data,
                          planning_internal::STGraphDebug* stGraphDebug) const;

  void RecordDebugInfo(const SpeedData& speed_data);
};
```

SpeedOptimizer类继承自Task类，实现了继承来的Execute()函数，又在Execute()中调用了其纯虚成员函数Process()。

```cpp
//task.h
class Task {
 public:
  explicit Task(const TaskConfig& config);

  virtual ~Task() = default;

  void SetName(const std::string& name) { name_ = name; }

  const std::string& Name() const;

  const TaskConfig& Config() const { return config_; }

  virtual apollo::common::Status Execute(
      Frame* frame, ReferenceLineInfo* reference_line_info);

 protected:
  Frame* frame_ = nullptr;
  ReferenceLineInfo* reference_line_info_ = nullptr;

  TaskConfig config_;
  std::string name_;
};
```

而DpStSpeedOptimizer类又继承了SpeedOptimizer类，并实现了Process()函数，在Process()中调用了成员函数SearchStGraph()开始使用动态规划算法进行速度规划。

~~~cpp
//modules\planning\tasks\optimizers\dp_st_speed\dp_st_speed_optimizer.cc
```
	bool DpStSpeedOptimizer::SearchStGraph(
    const StBoundaryMapper& boundary_mapper,
    const SpeedLimitDecider& speed_limit_decider, const PathData& path_data,
    SpeedData* speed_data, PathDecision* path_decision,
    STGraphDebug* st_graph_debug) const {
  std::vector<const StBoundary*> boundaries;
  for (auto* obstacle : path_decision->obstacles().Items()) {
    auto id = obstacle->Id();
    if (!obstacle->st_boundary().IsEmpty()) {
      if (obstacle->st_boundary().boundary_type() ==
          StBoundary::BoundaryType::KEEP_CLEAR) {
        path_decision->Find(id)->SetBlockingObstacle(false);
      } else {
        path_decision->Find(id)->SetBlockingObstacle(true);
      }
      boundaries.push_back(&obstacle->st_boundary());
    }
  }

  // step 2 perform graph search
  SpeedLimit speed_limit;
  if (!speed_limit_decider
           .GetSpeedLimits(path_decision->obstacles(), &speed_limit)
           .ok()) {
    AERROR << "Getting speed limits for dp st speed optimizer failed!";
    return false;
  }

  const double path_length = path_data.discretized_path().Length();
  StGraphData st_graph_data(boundaries, init_point_, speed_limit, path_length);

  DpStGraph st_graph(st_graph_data, dp_st_speed_config_,
                     reference_line_info_->path_decision()->obstacles().Items(),
                     init_point_, adc_sl_boundary_);

  if (!st_graph.Search(speed_data).ok()) {
    AERROR << "failed to search graph with dynamic programming.";
    RecordSTGraphDebug(st_graph_data, st_graph_debug);
    return false;
  }
  RecordSTGraphDebug(st_graph_data, st_graph_debug);
  return true;
}
```
~~~







