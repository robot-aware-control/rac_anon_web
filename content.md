----
## Motivation
Training visual control policies from scratch on a new robot typically requires generating large amounts of robot-specific data. Could we leverage data previously collected on another robot to reduce or even completely remove this need for robot-specific data? We propose a “robot-aware control” paradigm that achieves this by exploiting readily available knowledge about the robot. We then instantiate this in a robot-aware model-based RL policy by training modular dynamics models that couple a transferable, robot-agnostic world dynamics module with a robot- specific, potentially analytical, robot dynamics module. This also enables us to set up visual planning costs that separately consider the robot agent and the world. Our experiments on tabletop manipulation tasks with simulated and real robots demonstrate that these plug-in improvements dramatically boost the transferability of visual model-based RL policies, even permitting zero-shot transfer onto new robots for the very first time.

<br>

__TLDR:__ We integrate readily available knowledge about the robot into model-based policies to facilitate transfer to new robots.

----

## Robot-Aware Control (RAC)
Our key insight is that the robot and world can be factorized, and that building a robot-aware controller (RAC) that reflects such factorization will facilitate transfer to other robots. Concretely, we extend the visual foresight framework, a popular model based control method, by adding robot awareness in both the dynamics model and planning cost.
<br>
<div style="text-align: center;">
    <img class="b-lazy" src="img/wide_teaser.jpg" width="90%" >
    <figcaption>
        (Left) The modular visual dynamics model that factorizes into a learned world and analytical robot dynamics model. (Right) The unseen robot's analytical model is used to predict robot dynamics, permitting easy robot transfer.
    </figcaption>
</div>
<br>

First, we inject robot awareness into the dynamics model, training it to be invariant to robot appearance, by omitting the robot pixels in its input. This corresponds to a shared robot-invariant "world dynamics" model. We then propose to complete this dynamics model by pairing it with self-knowledge of analytical robot dynamics for each robot. This corresponds to a factorization of the dynamics into world and robot dynamics. Our experiments show that composing these two modules permits reliably transferring visual dynamics models even across robots that look and move very differently.

<br>
<div style="text-align: center;">
    <img class="b-lazy" src="img/wide_architecture.jpg" width="70%" >
    <figcaption>
        The visual dynamics architecture is composed of an analytical robot model, and the learned world model.
    </figcaption>
</div>
<br>

Next, we design a robot-aware planning cost over the separated robot and world pixels. We show that this not only allows visual task specifications to transfer from a source to a target robot, it even leads to gains on the source robot itself by allowing the controller to easily reason about the robot and its environment separately. This allows us to use goal images without robots, or even with humans.
<br>
<div style="text-align: center;">
    <img class="b-lazy"  src="img/cost_vis.jpg" width="70%" >
    <figcaption>
        We analyze the pixel cost and RA cost behavior on a goal image without a robot.
    </figcaption>
</div>
<br>
In the figure above, we illustrate how RA cost behaves correctly versus the conventional pixel-wise cost on a pushing trajectory and a no-robot goal image. The first row shows the feasible trajectory. The next two rows show the heatmaps (pixel-wise norm) between the image and goal for each cost. The heatmaps show high cost values in the robot region, while the RA cost heatmaps correctly have zero cost in the robot region. The plot on the right shows that the pixel cost fails to decrease as the trajectory progresses, while the RA cost correctly decreases. We will show in the experiments that pixel cost is unsuitable for optimization of goal images without robots or with humans.


---

## Prediction Experiment Visualizations
Next, we compare the outputs of the RA dynamics model against the baseline VF+State dynamics model when zero-shot transferred to a new robot. The RA model has better object dynamics prediction, less blurring, and correctly moves the test time robot. 
<div style="text-align: center;">
    <video class="b-lazy" width="70%" controls autoplay muted loop>
        <source src="img/prediction_rollouts.mp4" type="video/mp4">
    </video>
</div>

---

## Control Experiment Visualizations
We show goal-reaching episodes from the zero-shot pick-and-place and pushing control experiments, where we first train the dynamics models on a dataset of a single robot, and then zero-shot transfer the dynamics models to another robot. The models are used by the visual foresight controller to achieve the goal image using either the RA cost or pixel cost. Going forward we refer to all controllers by the names of the dynamics model and the cost, e.g., RA/RA for the full robot-aware controller. 

### Pick-and-place 
The models are trained on the WidowX robot, and then evaluated on the Fetch robot.
<table style="margin:0 auto">
    <tbody>
        <tr>
            <td style="width:45%; text-align: center">
                <img width="100%" src="img/pickplace/wxpp.png">
                <figcaption>
                    WidowX domain.
                </figcaption>
            </td>
            <td style="width:45%; text-align: center">
                <img width="100%" src="img/pickplace/fetchpp.png">
                <figcaption>
                    Fetch domain.
                </figcaption>
            </td>
        </tr>
    </tbody>
</table>
<br>
Here are some episode rollouts per method. The fully robot-aware policy gets the highest success rate.
<table>
    <tbody>
        <tr>
            <td style="width:25%; text-align: center">
            <b>RA/RA (ours): 8/20</b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/rac/exec_17.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/rac/exec_18.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/rac/exec_11.gif">
            </td>
        </tr>
        <tr>
            <td style="width:25%; text-align: center">
            <b>VF+State/RA: 4/20</b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/vf_ra/exec_0.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/vf_ra/exec_6.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/vf_ra/exec_8.gif">
            </td>
        </tr>
        <tr>
            <td style="width:25%; text-align: center">
            <b>VF+State/Pixel: 0/20</b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/vf_pixel/exec_0.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/vf_pixel/exec_4.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/vf_pixel/exec_9.gif">
            </td>
        </tr>
        <tr>
            <td style="width:25%; text-align: center">
            <b>RA/Pixel: 0/20</b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/ra_pixel/exec_12.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/ra_pixel/exec_16.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/ra_pixel/exec_18.gif">
            </td>
        </tr>
        <tr>
            <td style="width:25%; text-align: center">
            <b>CycleGAN: 0/20</b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/cyclegan/exec_0_crop.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/cyclegan/exec_6_crop.gif">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/pickplace/cyclegan/exec_8_crop.gif">
            </td>
        </tr>
    </tbody>
</table>


### Real Pushing
The models are trained on the WidowX robot, and then evaluated on the Franka robot. The fully robot-aware pipeline (RA/RA) has a success rate of 90%, whereas the baselines and ablations have below 20% success.

<div style="text-align: center;">
    <video class="b-lazy" width="100%" controls autoplay muted loop>
        <source src="img/control_rollouts.mp4" type="video/mp4">
    </video>
</div>
Here, we take a peek under the hood of the controllers by plotting the top 5 trajectories generated by each model during action optimization. We can see the baseline VF+State model fails to model the watermelon motion, whereas the RA model successfully models the downward motion of the watermelon. 
<div style="text-align: center;">
    <video class="b-lazy" width="80%" controls autoplay muted loop>
        <source src="img/cem.mp4" type="video/mp4">
    </video>
</div>
--- 
## Cost Experiment Visualizations
Here, we show that the RA cost can compute sensible distances for goal images with different agents such as humans, or goal images without any robots. Planners that compute the pixel cost for the same goal images result in failure. 
<div style="text-align: center;">
    <video class="b-lazy" width="80%" controls autoplay muted loop>
        <source src="img/human_goal.mp4" type="video/mp4">
    </video>
</div>

--- 
## CycleGAN Visualizations
First, we show some training data of the WidowX and Fetch for the CycleGAN. The CycleGAN learns to translate between the WidowX and Fetch images. Each robot's dataset is 12k images.
<table>
    <tbody>
        <tr>
            <td style="width:25%; text-align: center">
            <b>WidowX</b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/wx1.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/wx2.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/wx3.png">
            </td>
        </tr>
        <tr>
            <td style="width:25%; text-align: center">
            <b>Fetch </b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/fetch1.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/fetch2.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/fetch3.png">
            </td>
        </tr>
    </tbody>
</table>

<br>
After training, the CycleGAN produces visually realistic translations between the robots.
<table>
    <tbody>
        <tr>
            <td style="width:25%; text-align: center">
            <b>Input Image (Fetch)</b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/fetch_in1.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/fetch_in2.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/fetch_in3.png">
            </td>
        </tr>
        <tr>
            <td style="width:25%; text-align: center">
            <b>Translated Image (WidowX) </b>
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/wx_out1.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/wx_out2.png">
            </td>
            <td style="width:25%; text-align: center">
                <img width="100%" src="img/wx_out3.png">
            </td>
        </tr>
    </tbody>
</table>
<br>
However, despite being visually realistic, we found small errors in the translation that could affect pick-and-place performance of the baseline. For example, the CycleGAN tends to map the block orientation to the resting orientation. Here, in the Fetch environment, the block is rotated when grasped, resulting in an unstable grasp. However, when translated to the WidowX image, the block appears to be stably grasped.
<br>
<div style="text-align: center;">
    <img class="b-lazy" src="img/cycleganfail.png" width="60%" >
    <figcaption>
        (Left) The translated WidowX image appears to be stably holding a block. (Right) The true Fetch image has a rotated block.
    </figcaption>
</div>

---

## Citation
```
@inproceedings{
    anonymous2022know,
    title={Transferable Visual Control Policies Through Robot-Awareness},
    author={Anonymous},
    booktitle={Submitted to The Tenth International Conference on Learning Representations },
    year={2022},
    url={https://openreview.net/forum?id=o0ehFykKVtr},
    note={under review}
}
```