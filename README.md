[![Total alerts](https://img.shields.io/lgtm/alerts/g/mclerg00/ros_simulation_lab.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/mclerg00/ros_simulation_lab/alerts/)
# ROS Simulation Lab on Azure Pipelines

## Overview

After defining your robot behavior, it is important to ensure the robot running as expected as the project iterates.
This repository demonstrates how to run a autonomous car in the Gazebo simulator, test it on the rostest framework, and cloud-host the continuous integration on Azure Pipelines.

This lab uses the [software](https://github.com/Autonomous-Racing-PG/ar-tu-do) developed by the Autonomous Racing Project Group of [TU Dortmund](https://ls12-www.cs.tu-dortmund.de/daes/).
Credit goes to all the [contributors](https://github.com/Autonomous-Racing-PG/ar-tu-do/graphs/contributors).

## Objectives

* Run and observe the autonomous racecar simulation locally.

* Deploy and register a ROS on Windows GPU optimized virtual machine on Azure.

* Integrate a GitHub project with Azure Pipelines or GitHub Actions.

* Observe the simulation runs and test results on the pipeline.

## Prerequisites

* An Microsoft Azure account from https://portal.azure.com.

* An Azure DevOps account from https://dev.azure.com.

* A GitHub account from https://github.com.

This lab supports for both Windows and Linux.
For the Linux instructions, visit [here](./Linux.md).

## Exercise 1: Build And Run Autonomous Car Simulation Locally

1. Fork this repository into your GitHub account.
2. Follow this ROS Wiki [page](http://wiki.ros.org/Installation/Windows) to install ROS Melodic on Windows.
3. Open the ROS command prompt, and run the following to build the project.

```Batchfile
:: Clone the github project
git clone https://github.com/<your account>/ros_simulation_lab --recursive
cd ros_simulation_lab

:: install required components
vcpkg install sdl2:x64-windows
pip install circle-fit

:: build it
cd catkin_ws
catkin_make --use-ninja -DCMAKE_BUILD_TYPE=RELEASE
```

4. Run the autonomous car simulation.

```Batchfile
:: source the ROS devel space.
devel\setup.bat

:: run the application
roslaunch src\ar-tu-do\ros_ws\launch\gazebo.launch world:=racetrack mode_override:=2
```

This launch file runs a racecar in a simulated track in Gazebo and runs autonomous driving.

![simulation](docs/simulation.png)

## Exercise 2: Run ROSTest With Autonomous Car Simulation Locally

1. End the previous exercise and run the following rostest file:

```Batchfile
:: source the ROS devel space.
devel\setup.bat

:: run the rostest
rostest demo demo.test
```

This [`demo.test`](catkin_ws/src/demo/test/demo.test) runs the same task as the previous exercise but in headless mode.
Additonally, it runs a [`demo.py`](catkin_ws/src/demo/nodes/demo.py) node to kick off a validation on the latest lap time.
The [`demo`](catkin_ws/src/demo) package demonstrates an example how to organize your robot simulation with the rostest framework.
Now let's move this exercise to cloud-hosted environment with Azure Pipelines.

## Exercise 3: Provision Cloud CI Environment With Azure Pipelines

### Task 1: Prepare Permission For Azure DevOps Agent Pool

1. This [module](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops#permissions) guides you how to prepare permission for the agent pool.
2. Take a note of the personal access token (PAT).

### Task 2: Deploy Virtual Machine and Register as Azure DevOps Build Agent

This [`ROS on Azure with Windows VM`](https://azure.microsoft.com/en-us/resources/templates/ros-vm-windows/) is a Azure quickstart template to help setup an Azure virtual machine with ROS installed.

1. Navigate to the template. Click `Deploy to Azure`.
2. A form will be brought to you and here are some important parameters for this exercise.
   * **Virtual Machine Size**: Select `Standard_NV*` for GPU optimized virtual machine. This is required for Gazebo.
   * **Vm Image**: Select `Visual Studio 2019` for the required toolchain to build project.
   * **Pipeline Provider**: Select `AzurePipelines` to use Azure DevOps.
   * **Vsts Account**: This is your Azure DevOps organization name. For example, this is the `name` of `https://dev.azure.com/<name>`.
   * **Vsts Personal Access Token**: This is the PAT noted from the previous section.
   * **Vsts Pool Name**: Leave it to `Default` to match the pool name in this exercise.
   * **Enable Autologon**: Select `True` to run build agent in the interactive session. This is required for Gazebo.

   ![template](docs/template.png)


### Task 3: Integrate Your GitHub Projects With Azure Pipelines

1. Fork this repository into your GitHub account.
2. The [`Integrate Your GitHub Projects With Azure Pipelines`](https://www.azuredevopslabs.com/labs/azuredevops/github-integration/) guides you how to create a pipeline for a GitHub project in Task 1 & 2.
   Use your fork as the target repository.
3. Navigate to the "Existing Azure Pipelines YAML file" and select `azure-pipelines.yml`.
4. Now you should have a pipeline running (or ready to run).

### Task 4: Observe the Build Summary and Test Results

1. Click on a finished build and you will see a summary like:
   ![summary](docs/summary.png)

2. Check the `Related` and there is one artifact published, where you can find details logs for this run.
   In this example, you can also find the bag files for further analysis.
   ![logs](docs/logs.png)

3. Check the `Test and coverage` and you can find details test results by following the pass rate hyperlink.
   ![test_results](docs/test_results.png)

## Exercise 4: Provision Cloud CI Environment With GitHub Actions

### Task 1: Fork this GitHub Project

1. Fork this repository into your GitHub account.
2. Go to your forked repository and navigate to the `Actions` tab.
3. Make sure it is enabled by your permission.
   ![github-actions](docs/github-actions.png)

### Task 2: Prepare Permission For Self-Hosted GitHub Runner

1. Create a GitHub [personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) and select the scope of `repo`.
2. Take a note of the personal access token (PAT).

### Task 3: Deploy Virtual Machine and Register as Self-Hosted GitHub Runner

This [`ROS on Azure with Windows VM`](https://azure.microsoft.com/en-us/resources/templates/ros-vm-windows/) is a Azure quickstart template to help setup an Azure virtual machine with ROS installed.

1. Navigate to the template. Click `Deploy to Azure`.
2. A form will be brought to you and here are some important parameters for this exercise.
   * **Virtual Machine Size**: Select `Standard_NV*` for GPU optimized virtual machine. This is required for Gazebo.
   * **Vm Image**: Select `Visual Studio 2019` for the required toolchain to build project.
   * **Pipeline Provider**: Select `GitHubRunner` to use Azure DevOps.
   * **GitHub Repo**: This is your GitHub account and the repository name seperated by a forward slash. For example, `<your GitHub account>/ros_simulation_lab` is the value of this fork.
   * **GitHub Personal Access Token**: This is the PAT noted from the previous section.

### Task 4: Observe the GitHub Actions

1. Navigate to the `Actions` tab and make sure workflows are listed there.
   ![github-actions-summary](docs/github-actions-summary.png)

2. Push some changes to the fork.
3. Observe the runs of the workflows.
   And you can explore more on [GitHub Help](https://help.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow#about-workflows).

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
