add_subdirectory(gen)
add_subdirectory(maliput)

if(BUILD_TESTING)
  add_subdirectory(test)
endif()

add_library_with_exports(LIB_NAME drakeAutomotive SOURCE_FILES
  bicycle_car.cc
  calc_smooth_acceleration.cc
  create_trajectory_params.cc
  curve2.cc
  gen/bicycle_car_parameters.cc
  gen/bicycle_car_state.cc
  gen/driving_command.cc
  gen/euler_floating_joint_state.cc
  gen/idm_planner_parameters.cc
  gen/maliput_railcar_params.cc
  gen/maliput_railcar_state.cc
  gen/pure_pursuit_params.cc
  gen/simple_car_params.cc
  gen/simple_car_state.cc
  idm_controller.cc
  idm_planner.cc
  maliput/api/road_geometry.cc
  maliput/dragway/branch_point.cc
  maliput/dragway/junction.cc
  maliput/dragway/lane.cc
  maliput/dragway/road_geometry.cc
  maliput/dragway/segment.cc
  maliput/monolane/arc_lane.cc
  maliput/monolane/branch_point.cc
  maliput/monolane/builder.cc
  maliput/monolane/junction.cc
  maliput/monolane/lane.cc
  maliput/monolane/line_lane.cc
  maliput/monolane/loader.cc
  maliput/monolane/road_geometry.cc
  maliput/monolane/segment.cc
  maliput/utility/generate_obj.cc
  maliput/utility/generate_urdf.cc
  maliput_railcar.cc
  monolane_onramp_merge.cc
  pose_selector.cc
  pure_pursuit.cc
  pure_pursuit_controller.cc
  road_path.cc
  simple_car.cc
  simple_powertrain.cc
  trajectory_car.cc
  )
target_link_libraries(drakeAutomotive
  drakeCommon
  drakeSystemAnalysis
  drakeSystemFramework
  drakeSystemPrimitives
  drakeRendering
  fmt
  Eigen3::Eigen
  yaml-cpp
  )
drake_install_libraries(drakeAutomotive)
drake_install_headers(
  bicycle_car.h
  calc_smooth_acceleration.h
  create_trajectory_params.h
  curve2.h
  idm_controller.h
  idm_planner.h
  maliput_railcar.h
  monolane_onramp_merge.h
  road_path.h
  pose_selector.h
  pure_pursuit.h
  pure_pursuit_controller.h
  simple_car.h
  simple_car_to_euler_floating_joint.h
  simple_powertrain.h
  trajectory_car.h
  # N.B. The gen/*.h headers are installed by gen/CMakeLists.txt.
  )
drake_install_pkg_config_file(drake-automotive
  TARGET drakeAutomotive
  LIBS -ldrakeAutomotive
  REQUIRES
    drake-common
    drake-rbsystem
    drake-shapes
    drake-system-analysis
    drake-system-framework
    drake-system-primitives
    eigen3)

if(lcm_FOUND)
  include_directories(${PROJECT_SOURCE_DIR}/pod-build/lcmgen)

  add_library_with_exports(LIB_NAME drakeAutomotiveLcm SOURCE_FILES
    automotive_simulator.cc
    box_car_vis.cc
    car_vis.cc
    car_vis_applicator.cc
    gen/bicycle_car_parameters_translator.cc
    gen/bicycle_car_state_translator.cc
    gen/driving_command_translator.cc
    gen/euler_floating_joint_state_translator.cc
    gen/maliput_railcar_params_translator.cc
    gen/maliput_railcar_state_translator.cc
    gen/simple_car_params_translator.cc
    gen/simple_car_state_translator.cc
    prius_vis.cc
    )
  target_link_libraries(drakeAutomotiveLcm
    drakeAutomotive
    drakeLCMTypes
    drakeMultibodyParsers
    drakeRigidBodyPlant
    )
  drake_install_libraries(drakeAutomotiveLcm)
  drake_install_headers(
    automotive_simulator.h
    box_car_vis.h
    car_vis.h
    car_vis_applicator.h
    prius_vis.h
    # N.B. The gen/*.h headers are installed by gen/CMakeLists.txt.
    )
  drake_install_pkg_config_file(drake-automotive-lcm
    TARGET drakeAutomotiveLcm
    LIBS -ldrakeAutomotiveLcm
    REQUIRES
      drake-automotive
      drake-rigid-body-plant)

  add_executable(automotive_demo automotive_demo.cc)
  target_link_libraries(automotive_demo
      drakeAutomotiveLcm
      drakeMaliputDragway)

  add_executable(car_sim_lcm car_sim_lcm.cc car_sim_lcm_common.cc)
  target_link_libraries(car_sim_lcm
      drakeAutomotiveLcm
      drakeLcmSystem
      drakeRigidBodyPlant
      drakeSystemAnalysis
      drakeSystemControllers)
  drake_add_test(NAME car_sim_lcm COMMAND car_sim_lcm --simulation_sec 0.01)

  # TODO(liang.fok) Enable the following unit test once the build system is able
  # to automatically generate the speed_bump.obj file from the speed_bump.yaml
  # file.
  # drake_add_test(NAME car_sim_lcm COMMAND car_sim_lcm --with_speed_bump true --simulation_sec 0.01)
endif()
