---
author: TK
---





今天练习了如何给一个电机编程用手柄控制正反转，另外加了一些底盘内容



Robot.java

```java
/*----------------------------------------------------------------------------*/
/* Copyright (c) 2017-2018 FIRST. All Rights Reserved.                        */
/* Open Source Software - may be modified and shared by FRC teams. The code   */
/* must be accompanied by the FIRST BSD license file in the root directory of */
/* the project.                                                               */
/*----------------------------------------------------------------------------*/

package frc.robot;

/*** import 需要用到的库文件 ***/
import com.ctre.phoenix.motorcontrol.ControlMode;
import com.ctre.phoenix.motorcontrol.can.WPI_VictorSPX;
import edu.wpi.first.wpilibj.TimedRobot;
import edu.wpi.first.wpilibj.smartdashboard.SendableChooser;
import edu.wpi.first.wpilibj.smartdashboard.SmartDashboard;
import edu.wpi.first.wpilibj.VictorSP;
import edu.wpi.first.wpilibj.XboxController;
import edu.wpi.first.wpilibj.GenericHID.Hand;

/**
 * The VM is configured to automatically run this class, and to call the
 * functions corresponding to each mode, as described in the IterativeRobot
 * documentation. If you change the name of this class or the package after
 * creating this project, you must also update the build.gradle file in the
 * project.
 */
public class Robot extends TimedRobot {
  /*** 下面四行是用来定义自动阶段的程序选择的，不用管 ***/
  private static final String kDefaultAuto = "Default";
  private static final String kCustomAuto = "My Auto";
  private String m_autoSelected;
  private final SendableChooser<String> m_chooser = new SendableChooser<>();

  /*** 声明手柄变量，注意类型为 XboxController，括号中的数字对应 DriverStation 中
   * 手柄的序号，第一个是 0 端口 ***/
  private XboxController joystick = new XboxController(0);

  /*** 声明一个 VictorSP 控制器控制的电机的变量，这里还不进行初始化，只是声明 ***/
  private VictorSP motor;

  /*** 声明四个底盘电机，同理不初始化 ***/
  private WPI_VictorSPX left_motor_0;
  private WPI_VictorSPX left_motor_1;
  private WPI_VictorSPX right_motor_0;
  private WPI_VictorSPX right_motor_1;

  /**
   * This function is run when the robot is first started up and should be
   * used for any initialization code.
   */
  /*** 这个方法在机器人通电或者传完程序的时候被执行一次 ***/
  @Override
  public void robotInit() {
    /*** 下面三个仍然是自动阶段的东西，不用管 ***/
    m_chooser.setDefaultOption("Default Auto", kDefaultAuto);
    m_chooser.addOption("My Auto", kCustomAuto);
    SmartDashboard.putData("Auto choices", m_chooser);

    /*** 初始化电机对象，后面的 2 代表 PWM接口 2号端口 ***/
    motor = new VictorSP(2);

    /*** 初始化底盘电机对象，是用 CAN 线控制的，后面的数字代表 CAN 线上控制器的
     * ID （可以通过 Phoenix Tuner 软件查看） ***/
    left_motor_0 = new WPI_VictorSPX(10);
    left_motor_1 = new WPI_VictorSPX(11);
    right_motor_0 = new WPI_VictorSPX(12);
    right_motor_1 = new WPI_VictorSPX(13);
    
  }

  /**
   * This function is called every robot packet, no matter the mode. Use
   * this for items like diagnostics that you want ran during disabled,
   * autonomous, teleoperated and test.
   *
   * <p>This runs after the mode specific periodic functions, but before
   * LiveWindow and SmartDashboard integrated updating.
   */
  /*** 这个方法会被一直循环执行，不论模式。一般在这里只放一些 Log 类的程序 ***/
  @Override
  public void robotPeriodic() {
  }

  /*** ↓↓↓↓↓↓↓↓ 这一块不用看 ↓↓↓↓↓↓↓↓ ***/

  /**
   * This autonomous (along with the chooser code above) shows how to select
   * between different autonomous modes using the dashboard. The sendable
   * chooser code works with the Java SmartDashboard. If you prefer the
   * LabVIEW Dashboard, remove all of the chooser code and uncomment the
   * getString line to get the auto name from the text box below the Gyro
   *
   * <p>You can add additional auto modes by adding additional comparisons to
   * the switch structure below with additional strings. If using the
   * SendableChooser make sure to add them to the chooser code above as well.
   */
  @Override
  public void autonomousInit() {
    m_autoSelected = m_chooser.getSelected();
    // autoSelected = SmartDashboard.getString("Auto Selector",
    // defaultAuto);
    System.out.println("Auto selected: " + m_autoSelected);
  }

  /**
   * This function is called periodically during autonomous.
   */
  @Override
  public void autonomousPeriodic() {
    switch (m_autoSelected) {
      case kCustomAuto:
        // Put custom auto code here
        break;
      case kDefaultAuto:
      default:
        // Put default auto code here
        break;
    }
  }

  /*** ↑↑↑↑↑↑↑↑↑ 这一块不用看 ↑↑↑↑↑↑↑↑↑ ***/

  /**
   * This function is called periodically during operator control.
   */
  /*** 这个方法在机器人手动操控阶段【循环执行】 ***/
  @Override
  public void teleopPeriodic() {

    /*** .set(double speed) 设定电机速度，将 Trigger 的 0~1 转换为 -1~1 ***/
    motor.set(joystick.getTriggerAxis(Hand.kLeft) - joystick.getTriggerAxis(Hand.kRight));

    
    /*** @梁胜凯 提供的底盘代码 ***/
    if (joystick.getY(Hand.kLeft) > 0.2 || joystick.getY(Hand.kLeft) < -0.2 ) {
      left_motor_0.set(ControlMode.PercentOutput, -0.5 * joystick.getY(Hand.kLeft));
      left_motor_1.set(ControlMode.PercentOutput, -0.5 * joystick.getY(Hand.kLeft));
      right_motor_0.set(ControlMode.PercentOutput, 0.5 * joystick.getY(Hand.kLeft));
      right_motor_1.set(ControlMode.PercentOutput, 0.5 * joystick.getY(Hand.kLeft));
    } else if (joystick.getX(Hand.kLeft) > 0.2 || joystick.getX(Hand.kLeft) < -0.2 ) {
      left_motor_0.set(ControlMode.PercentOutput, 0.5 * joystick.getX(Hand.kLeft));
      left_motor_1.set(ControlMode.PercentOutput, 0.5 * joystick.getX(Hand.kLeft));
      right_motor_0.set(ControlMode.PercentOutput, 0.5 * joystick.getX(Hand.kLeft));
      right_motor_1.set(ControlMode.PercentOutput, 0.5 * joystick.getX(Hand.kLeft));
    } else {
      left_motor_0.set(0);
      left_motor_1.set(0);
      right_motor_0.set(0);
      right_motor_1.set(0);
    }

    /*** 底盘的另一种写法，if (false)只是让这段程序不执行，以免和上文冲突，相
     * 当于注释掉。 VS Code 会报错 Dead Code，请忽略 ***/
    if (false) {
      left_motor_0.set(ControlMode.PercentOutput, -0.5 * (joystick.getY(Hand.kLeft) + joystick.getX(Hand.kLeft)));
      left_motor_1.set(ControlMode.PercentOutput, -0.5 * (joystick.getY(Hand.kLeft) + joystick.getX(Hand.kLeft)));
      right_motor_0.set(ControlMode.PercentOutput, 0.5 * (joystick.getY(Hand.kLeft) + joystick.getX(Hand.kLeft)));
      right_motor_0.set(ControlMode.PercentOutput, 0.5 * (joystick.getY(Hand.kLeft) + joystick.getX(Hand.kLeft)));
    }
  }

  /**
   * This function is called periodically during test mode.
   */
  /*** 废物方法.... ***/
  @Override
  public void testPeriodic() {

  }
}

```

