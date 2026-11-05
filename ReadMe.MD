<div align="center">

# SW Robotics Minnetrials Code

### Command-Based Java Guide For New Code Team Members

</div>

---

> **Purpose:** This repository contains the command-based Java code for the SW Robotics Minnetrials robot. This document explains our code structure, subsystem format, NetworkTables tuning, SmartDashboard logging, commands, controller bindings, autonomous routines, and how to use them to make your own robot.

---

# Project Structure

```text
src/main/java/com/swrobotics
│
├── lib/
│   ├── ctre/...
│   └── net/...
├── robot/
│   │
│   ├── commands/
│   │   ├── AutonomousCommands.java
│   │   └── DriveCommands.java
│   │
│   ├── config/
│   │   ├── FieldConstants.java
│   │   ├── IOAllocation.java
│   │   └── RobotConstants.java
│   │
│   ├── control/
│   │   └── ControlBoard.java
│   │
│   ├── logging/
│   │   └── FieldView.java
│   │
│   ├── subsystems/
│   │   ├── ExampleArmSubsystem.java
│   │   ├── ExampleShooterSubsystem.java
│   │   └── drive/
│   │       └── DriveSubsystem.java
│   ├── Main.java
│   ├── Robot.java
│   └── RobotContainer.java
│   
└──</code></pre>

    <table>
        <thead>
            <tr>
                <th>Location</th>
                <th>Responsibility</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>lib/</code></td>
                <td>This has information that allows your network tables to run. don't really need to edit</td>
            </tr>
            <tr>
                <td><code>commands/</code></td>
                <td>Contains reusable actions and autonomous sequences: for example, autoalign. edit this</td>
            </tr>
            <tr>
                <td><code>config/</code></td>
                <td>Contains global constants for field positions, Motor IDs, and Robot constants. edit this</td>
            </tr>
            <tr>
                <td><code>control/</code></td>
                <td>Maps controls, default commands, button bindings. edit this</td>
            </tr>
            <tr>
                <td><code>logging/</code></td>
                <td>Used so you can see where your robot is on the field in sim and real game.</td>
            </tr>
            <tr>
                <td><code>subsystems/</code></td>
                <td>Controls individual physical mechanisms. create new ones.</td>
            </tr>
            <tr>
                <td><code>Main.java</code></td>
                <td>Starts the robot program: no need to mess with.</td>
            </tr>
            <tr>
                <td><code>Robot.java</code></td>
                <td>Runs robot lifecycle methods and the command scheduler: not really needed to mess with</td>
            </tr>
            <tr>
                <td><code>RobotContainer.java</code></td>
                <td>Creates subsystems, and autonomous routines: you need to use this</td>
            </tr>
        </tbody>
    </table>

    <h1>Command-Based Overview</h1>

    <p>This project uses the WPILib command-based framework.</p>

    <p>A <strong>subsystem</strong> represents a physical robot mechanism. A <strong>command</strong> represents an action the robot can perform with one or more mechanisms.</p>

    <table>
        <thead>
            <tr>
                <th>Type</th>
                <th>Description</th>
                <th>Examples</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Subsystem</td>
                <td>Owns hardware and mechanism behavior</td>
                <td>Drivebase, arm, shooter, intake</td>
            </tr>
            <tr>
                <td>Command</td>
                <td>Requests a robot action</td>
                <td>Drive with joystick, move arm down, spin shooter</td>
            </tr>
            <tr>
                <td>State</td>
                <td>Describes the intended mode of a subsystem</td>
                <td><code>IDLE</code>, <code>SHOOT</code>, <code>UP</code>, <code>DOWN</code></td>
            </tr>
        </tbody>
    </table>

    <p>The expected flow is:</p>

<pre><code>Controller Input
      ↓
ControlBoard uses the input to tell a command to run
      ↓
Command runs
      ↓
Subsystem State changes
      ↓
periodic() updates
      ↓
Motor/Sensor Control and SmartDashboard Logging update as well</code></pre>

    <p>A subsystem owns its motors, sensors, configuration, states, NetworkTables entries, and SmartDashboard logs.</p>

    <p>A command should request a subsystem behavior. It should not contain random motor-control code that belongs inside a subsystem.</p>

    <hr>

    <h1>Subsystem Format</h1>

    <p>Every physical robot mechanism should have a subsystem.</p>

    <p>Examples:</p>

<pre><code>DriveSubsystem
ExampleArmSubsystem
ExampleShooterSubsystem
IntakeSubsystem
IndexerSubsystem
ClimberSubsystem</code></pre>

    <p>A subsystem should usually follow this organization:</p>

<pre><code><span class="custom-blue-text">public class</span> ExampleSubsystem <span class="custom-blue-text">extends</span> SubsystemBase {
    // 1. State enum

    // 2. Hard-coded hardware constants

    // 3. NetworkTables tunables

    // 4. Hardware objects and control requests

    // 5. Target state

    // 6. Constructor and hardware configuration

    // 7. periodic() motor control and SmartDashboard logs

    // 8. Public helper methods

    // 9. Command factory methods
}</code></pre>

    <h2>Basic subsystem template</h2>

<pre><code>package com.swrobotics.robot.subsystems;

import edu.wpi.first.wpilibj2.command.SubsystemBase;

public class ExampleSubsystem extends SubsystemBase {
    public enum State {
        IDLE,
        ACTIVE
    }

    // Hardware constants
    private static final int kMotorID = 8;

    // Current requested mechanism behavior
    private State targetState = State.IDLE;

    public ExampleSubsystem() {
        // Create and configure hardware here.
    }

    @Override
    public void periodic() {
        switch (targetState) {
            case IDLE:
                // Stop or hold the mechanism.
                break;

            case ACTIVE:
                // Run the mechanism.
                break;
        }
    }

    public void setTargetState(State state) {
        targetState = state;
    }
}</code></pre>

    <h2>State rules</h2>

    <p>States should describe a real mechanism behavior.</p>

    <p>Good state names:</p>

<pre><code>IDLE
INTAKE
OUTTAKE
FEED
HOLD
UP
DOWN
STOW
DEPLOY
SHOOT
REVERSE
CLIMB</code></pre>

    <p>Avoid state names based on code details or controller buttons.</p>

    <p>Avoid:</p>

<pre><code>BUTTON_A
MOTOR_50_PERCENT
RUN_MOTOR
TEST_2
DO_THING
MOTOR GO
shoot &lt;- this one is bad because it isn't all caps.</code></pre>

    <hr>

    <h1>Constants and Tunables</h1>

    <p>Every number in robot code should have a name and a reason.</p>

    <p>Don't leave unexplained numbers directly in motor-control calls; otherwise, it will be hard to tell what they mean or are.</p>

    <p>Bad:</p>

<pre><code>motor.setControl(voltageControl.withOutput(6.0));</code></pre>

    <p>Better:</p>

<pre><code>private final NTEntry&lt;Double&gt; intakeVoltage = new NTDouble("Intake/IntakeVoltage", 6.0).setPersistent();</code></pre>

<pre><code>motor.setControl(voltageControl.withOutput(intakeVoltage.get()));</code></pre>

    <h2>Hard-coded constants</h2>

    <p>Use hard-coded values for physical hardware configuration and fixed design decisions.</p>

<pre><code>private static final int kMotorID = 5;
private static final InvertedValue kMotorInverted = InvertedValue.Clockwise_Positive;
private static final NeutralModeValue kNeutralMode = NeutralModeValue.Brake;</code></pre>

    <p>Use hard-coded constants for:</p>

    <ul>
        <li>CAN IDs</li>
        <li>DIO ports</li>
        <li>PWM ports</li>
        <li>Motor inversion</li>
        <li>Brake/coast mode</li>
        <li>Fixed physical configuration</li>
        <li>Static safety choices</li>
    </ul>

    <p><strong>ALWAYS make CAN IDs hard-coded values.</strong></p>

    <h2>NetworkTables tunables</h2>

    <p>Use NetworkTables entries for numbers that may change during robot testing, calibration, driver practice, or mechanism tuning.</p>

<pre><code>private final NTEntry&lt;Double&gt; kP = new NTDouble("ExampleArm/kP", 2.0).setPersistent();
private final NTEntry&lt;Double&gt; kI = new NTDouble("ExampleArm/kI", 0.0).setPersistent();
private final NTEntry&lt;Double&gt; kD = new NTDouble("ExampleArm/kD", 0.1).setPersistent();</code></pre>

    <p>Use NetworkTables values for:</p>

    <ul>
        <li>PID values</li>
        <li>Arm setpoints</li>
        <li>Shooter speed targets</li>
        <li>Drive speed scaling</li>
        <li>Joystick deadband</li>
        <li>Motion Magic velocity and acceleration</li>
        <li>Current limits that may need testing adjustments</li>
        <li>Vision aiming constants</li>
    </ul>

    <h2>PID overview</h2>

    <table>
        <thead>
            <tr>
                <th>Term</th>
                <th>Full Name</th>
                <th>Purpose</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>kP</code></td>
                <td>Proportional</td>
                <td>Responds to the current error</td>
            </tr>
            <tr>
                <td><code>kI</code></td>
                <td>Integral</td>
                <td>Corrects error that remains for a long time</td>
            </tr>
            <tr>
                <td><code>kD</code></td>
                <td>Derivative</td>
                <td>Reduces overshoot and oscillation</td>
            </tr>
        </tbody>
    </table>

    <p>Safe tuning order:</p>

    <ol>
        <li>Start with <code>kI = 0.0</code> and <code>kD = 0.0</code>.</li>
        <li>Increase <code>kP</code> until the mechanism responds well.</li>
        <li>Add a small <code>kD</code> only if the mechanism overshoots or oscillates.</li>
        <li>Add a very small <code>kI</code> only if a consistent remaining error cannot be fixed another way.</li>
    </ol>

    <hr>

    <h1>SmartDashboard Logging</h1>

    <p>Every subsystem should report useful information to SmartDashboard.</p>

    <p>The goal of dashboard logging is to answer:</p>

    <ul>
        <li>What state is the subsystem in?</li>
        <li>What target did the code request?</li>
        <li>What does the sensor actually read?</li>
        <li>Is the mechanism at the target?</li>
        <li>Is the motor drawing too much current?</li>
        <li>Is the motor getting too hot?</li>
    </ul>

    <h2>Log naming format</h2>

    <p>Use this naming pattern:</p>

<pre><code>SubsystemName/ValueName</code></pre>

    <p>Examples:</p>

<pre><code>SmartDashboard.putString("ExampleArm/State", targetState.name());
SmartDashboard.putNumber("ExampleArm/TargetPositionRotations", targetPositionRotations);
SmartDashboard.putNumber("ExampleArm/ActualPositionRotations", armMotor.getPosition().getValueAsDouble());
SmartDashboard.putNumber("ExampleArm/MotorVoltage", armMotor.getMotorVoltage().getValueAsDouble());
SmartDashboard.putNumber("ExampleArm/TemperatureCelsius", armMotor.getDeviceTemp().getValueAsDouble());</code></pre>

    <h2>Recommended TalonFX logs</h2>

<pre><code>SmartDashboard.putString("ExampleArm/State", targetState.name());
SmartDashboard.putNumber("Subsystem/PositionRotations", motor.getPosition().getValueAsDouble());
SmartDashboard.putNumber("Subsystem/VelocityRPS", motor.getVelocity().getValueAsDouble());
SmartDashboard.putNumber("Subsystem/MotorVoltage", motor.getMotorVoltage().getValueAsDouble());
SmartDashboard.putNumber("Subsystem/TemperatureCelsius", motor.getDeviceTemp().getValueAsDouble());</code></pre>

    <hr>

    <h1>Adding a New Subsystem</h1>

    <p>Follow this process whenever you add a new mechanism.</p>

    <h2>1. Create the file</h2>

    <p>For an intake:</p>

<pre><code>src/main/java/com/swrobotics/robot/subsystems/intake/IntakeSubsystem.java</code></pre>

<pre><code>package com.swrobotics.robot.subsystems.intake;

import edu.wpi.first.wpilibj2.command.SubsystemBase;

public class IntakeSubsystem extends SubsystemBase {
}</code></pre>

    <h2>2. Add states</h2>

<pre><code>public enum State {
    IDLE,
    INTAKE,
    OUTTAKE
}</code></pre>

    <h2>3. Add hardware constants</h2>

<pre><code>private static final int kMotorID = 8;
private static final InvertedValue kMotorInverted = InvertedValue.Clockwise_Positive;
private static final NeutralModeValue kNeutralMode = NeutralModeValue.Coast;</code></pre>

    <h2>4. Add tunable values</h2>

<pre><code>private final NTEntry&lt;Double&gt; intakeVoltage = new NTDouble("Intake/IntakeVoltage", 6.0).setPersistent();
private final NTEntry&lt;Double&gt; outtakeVoltage = new NTDouble("Intake/OuttakeVoltage", -4.0).setPersistent();</code></pre>

    <h2>5. Create motor hardware</h2>

<pre><code>private final TalonFX motor = new TalonFX(kMotorID);
private final VoltageOut voltageControl = new VoltageOut(0.0);</code></pre>

    <h2>6. Control the mechanism in <code>periodic()</code></h2>

<pre><code>@Override
public void periodic() {
    switch (targetState) {
        case IDLE:
            motor.stopMotor();
            break;

        case INTAKE:
            motor.setControl(voltageControl.withOutput(intakeVoltage.get()));
            break;

        case OUTTAKE:
            motor.setControl(voltageControl.withOutput(outtakeVoltage.get()));
            break;
    }
}</code></pre>

    <h2>7. Add SmartDashboard logs</h2>

<pre><code>SmartDashboard.putString("Intake/State", targetState.name());
SmartDashboard.putNumber("Intake/VelocityRPS", motor.getVelocity().getValueAsDouble());
SmartDashboard.putNumber("Intake/SupplyCurrentAmps", motor.getSupplyCurrent().getValueAsDouble());
SmartDashboard.putNumber("Intake/TemperatureCelsius", motor.getDeviceTemp().getValueAsDouble());</code></pre>

    <h2>8. Add state methods and a command</h2>

<pre><code>public void setTargetState(State state) {
    targetState = state;
}

public Command commandSetState(State state) {
    return Commands.run(() -&gt; setTargetState(state), this).withName("Intake " + state.name());
}</code></pre>

    <h2>9. Create it in <code>RobotContainer</code></h2>

<pre><code>public final IntakeSubsystem intake = new IntakeSubsystem();</code></pre>

    <h2>10. Bind a controller input</h2>

<pre><code>operator.rightTrigger().whileTrue(robot.intake.commandSetState(IntakeSubsystem.State.INTAKE));</code></pre>

    <hr>

    <h1>Command Format</h1>

    <p>Commands represent robot actions.</p>

    <p>Use a command when you need to:</p>

    <ul>
        <li>Change a subsystem state</li>
        <li>Coordinate multiple subsystems</li>
        <li>Run a mechanism for a limited time</li>
        <li>Wait for a sensor</li>
        <li>Write an autonomous routine</li>
        <li>Build a reusable robot action</li>
    </ul>

    <h2>Command patterns</h2>

    <table>
        <thead>
            <tr>
                <th>Pattern</th>
                <th>Use when</th>
                <th>Example</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>Commands.run(...)</code></td>
                <td>The action continues until interrupted</td>
                <td>Keep shooter spinning while held</td>
            </tr>
            <tr>
                <td><code>Commands.runOnce(...)</code></td>
                <td>The action occurs once and then finishes</td>
                <td>Set arm state to <code>DOWN</code></td>
            </tr>
            <tr>
                <td><code>.withTimeout(...)</code></td>
                <td>Stop after a fixed amount of time</td>
                <td>Drive forward for 2 seconds</td>
            </tr>
            <tr>
                <td><code>.until(...)</code></td>
                <td>Stop when a condition becomes true</td>
                <td>Intake until a sensor detects a note</td>
            </tr>
            <tr>
                <td><code>Commands.sequence(...)</code></td>
                <td>Run actions in order</td>
                <td>Drive, shoot, then stop</td>
            </tr>
            <tr>
                <td><code>Commands.parallel(...)</code></td>
                <td>Run actions at the same time</td>
                <td>Drive while raising an arm</td>
            </tr>
        </tbody>
    </table>

    <h2>Simple state command</h2>

    <p>Subsystem commands should usually be short:</p>

<pre><code>public Command commandSetState(State state) {
    return Commands.run(() -&gt; setTargetState(state), this).withName("Shooter " + state.name());
}</code></pre>

    <hr>

    <h1>Example Drive Command</h1>

    <p>Create:</p>

<pre><code>src/main/java/com/swrobotics/robot/commands/ExampleDriveCommand.java</code></pre>

<pre><code>package com.swrobotics.robot.commands;

import java.util.function.DoubleSupplier;

import com.swrobotics.robot.subsystems.drive.DriveSubsystem;

import edu.wpi.first.math.MathUtil;
import edu.wpi.first.wpilibj2.command.Command;

public class ExampleDriveCommand extends Command {
    private static final double kDeadband = 0.10;

    private final DriveSubsystem drive;
    private final DoubleSupplier forwardSupplier;
    private final DoubleSupplier turnSupplier;

    public ExampleDriveCommand(DriveSubsystem drive, DoubleSupplier forwardSupplier, DoubleSupplier turnSupplier) {
        this.drive = drive;
        this.forwardSupplier = forwardSupplier;
        this.turnSupplier = turnSupplier;

        addRequirements(drive);
    }

    @Override
    public void execute() {
        double forward = MathUtil.applyDeadband(forwardSupplier.getAsDouble(), kDeadband);
        double turn = MathUtil.applyDeadband(turnSupplier.getAsDouble(), kDeadband);

        drive.arcadeDrive(forward, turn);
    }

    @Override
    public void end(boolean interrupted) {
        drive.stop();
    }

    @Override
    public boolean isFinished() {
        return false;
    }
}</code></pre>

    <p>Your <code>DriveSubsystem</code> needs these public methods:</p>

<pre><code>public void arcadeDrive(double forward, double turn) {
    double leftOutput = forward + turn;
    double rightOutput = forward - turn;

    leftMotor1.set(ControlMode.PercentOutput, leftOutput);
    leftMotor2.set(ControlMode.PercentOutput, leftOutput);
    rightMotor1.set(ControlMode.PercentOutput, rightOutput);
    rightMotor2.set(ControlMode.PercentOutput, rightOutput);
}

public void stop() {
    leftMotor1.set(ControlMode.PercentOutput, 0.0);
    leftMotor2.set(ControlMode.PercentOutput, 0.0);
    rightMotor1.set(ControlMode.PercentOutput, 0.0);
    rightMotor2.set(ControlMode.PercentOutput, 0.0);
}</code></pre>

    <p>Set it as the default drive command:</p>

<pre><code>drive.setDefaultCommand(new ExampleDriveCommand(drive, () -&gt; controls.driver.getLeftY(), () -&gt; controls.driver.getRightX()));</code></pre>

    <p>This command runs continuously during teleoperated driving. It reads joystick values, applies deadband, and sends arcade-drive outputs to the drivetrain.</p>

    <hr>

    <h1>Example Autonomous Command</h1>

    <p>Create:</p>

<pre><code>src/main/java/com/swrobotics/robot/commands/ExampleAutonomous.java</code></pre>

<pre><code>package com.swrobotics.robot.commands;

import com.swrobotics.robot.subsystems.ExampleShooterSubsystem;
import com.swrobotics.robot.subsystems.drive.DriveSubsystem;

import edu.wpi.first.wpilibj2.command.Command;
import edu.wpi.first.wpilibj2.command.Commands;

public final class ExampleAutonomous {
    private ExampleAutonomous() {
    }

    public static Command create(DriveSubsystem drive, ExampleShooterSubsystem shooter) {
        return Commands.sequence(
                shooter.commandSetState(ExampleShooterSubsystem.State.IDLE).withTimeout(0.02),
                drive.driveForTime(0.40, 0.0, 2.0),
                drive.stopCommand(),
                shooter.commandSetState(ExampleShooterSubsystem.State.SHOOT).withTimeout(1.5),
                shooter.commandSetState(ExampleShooterSubsystem.State.IDLE).withTimeout(0.02)
        ).withName("Example Autonomous");
    }
}</code></pre>

    <p>The routine does this:</p>

    <table>
        <thead>
            <tr>
                <th>Step</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>1</td>
                <td>Make sure the shooter is idle</td>
            </tr>
            <tr>
                <td>2</td>
                <td>Drive forward at 40% output for 2 seconds</td>
            </tr>
            <tr>
                <td>3</td>
                <td>Stop the drivetrain</td>
            </tr>
            <tr>
                <td>4</td>
                <td>Spin the shooter for 1.5 seconds</td>
            </tr>
            <tr>
                <td>5</td>
                <td>Return the shooter to idle</td>
            </tr>
        </tbody>
    </table>

    <p>Add these methods to <code>DriveSubsystem</code>:</p>

<pre><code>public Command driveForTime(double forwardOutput, double turnOutput, double seconds) {
    return run(() -&gt; arcadeDrive(forwardOutput, turnOutput)).withTimeout(seconds).finallyDo(interrupted -&gt; stop()).withName("Drive For Time");
}

public Command stopCommand() {
    return runOnce(this::stop).withName("Drive Stop");
}</code></pre>

    <p>Add this to <code>RobotContainer.java</code>:</p>

<pre><code>import com.swrobotics.robot.commands.ExampleAutonomous;

public Command getAutonomousCommand() {
    return ExampleAutonomous.create(drive, exampleShooter);
}</code></pre>

    <hr>

    <h1>Controller Bindings</h1>

    <p>Controller bindings belong in:</p>

<pre><code>control/ControlBoard.java</code></pre>

    <p>Example:</p>

<pre><code>private void configureControls() {
    robot.drive.setDefaultCommand(new ExampleDriveCommand(robot.drive, () -&gt; driver.getLeftY(), () -&gt; driver.getRightX()));

    operator.a().onTrue(robot.exampleArm.commandSetState(ExampleArmSubsystem.State.UP));
    operator.b().onTrue(robot.exampleArm.commandSetState(ExampleArmSubsystem.State.DOWN));

    operator.rightBumper().whileTrue(robot.exampleShooter.commandSetState(ExampleShooterSubsystem.State.SHOOT));
    operator.leftBumper().whileTrue(robot.exampleShooter.commandSetState(ExampleShooterSubsystem.State.REVERSE));
}</code></pre>

    <h2>Button binding choices</h2>

    <table>
        <thead>
            <tr>
                <th>Binding</th>
                <th>Use case</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>onTrue()</code></td>
                <td>Change a state once when pressed</td>
            </tr>
            <tr>
                <td><code>whileTrue()</code></td>
                <td>Run a command only while a button is held</td>
            </tr>
            <tr>
                <td><code>onFalse()</code></td>
                <td>Run a command when a button is released</td>
            </tr>
            <tr>
                <td><code>toggleOnTrue()</code></td>
                <td>Toggle a command on and off</td>
            </tr>
        </tbody>
    </table>

    <p>Use <code>onTrue()</code> for arm positions because the arm should stay at the requested position.</p>

    <p>Use <code>whileTrue()</code> for a shooter when the shooter should stop immediately after the driver releases the button.</p>

    <hr>

    <h1>Code Style Rules</h1>

    <h2>Naming</h2>

<pre><code>private static final int kMotorID = 5;
private final TalonFX shooterMotor = new TalonFX(kMotorID);
private final NTEntry&lt;Double&gt; shootVelocityRps = new NTDouble("Shooter/ShootVelocityRPS", 60.0).setPersistent();</code></pre>

    <table>
        <thead>
            <tr>
                <th>Item</th>
                <th>Style</th>
                <th>Example</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Class</td>
                <td>PascalCase</td>
                <td><code>ExampleShooterSubsystem</code></td>
            </tr>
            <tr>
                <td>Method</td>
                <td>camelCase</td>
                <td><code>setTargetState()</code></td>
            </tr>
            <tr>
                <td>Variable</td>
                <td>camelCase</td>
                <td><code>targetVelocityRps</code></td>
            </tr>
            <tr>
                <td>Hard constant</td>
                <td><code>k</code> + PascalCase</td>
                <td><code>kMotorID</code></td>
            </tr>
            <tr>
                <td>NetworkTables entry</td>
                <td>camelCase</td>
                <td><code>shootVelocityRps</code></td>
            </tr>
            <tr>
                <td>Enum value</td>
                <td>UPPER_SNAKE_CASE</td>
                <td><code>SLOW_SHOOT</code></td>
            </tr>
            <tr>
                <td>Dashboard key</td>
                <td><code>Subsystem/ValueName</code></td>
                <td><code>ExampleShooter/TopVelocityRPS</code></td>
            </tr>
        </tbody>
    </table>

    <h2>Keep responsibilities separate</h2>

    <table>
        <thead>
            <tr>
                <th>Location</th>
                <th>Should contain</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Subsystem</td>
                <td>Motors, sensors, subsystem states, motor configuration, periodic control, dashboard logs</td>
            </tr>
            <tr>
                <td>Command</td>
                <td>A requested action, command sequence, timing, multi-subsystem coordination</td>
            </tr>
            <tr>
                <td>ControlBoard</td>
                <td>Controller mappings and button bindings</td>
            </tr>
            <tr>
                <td>RobotContainer</td>
                <td>Subsystem construction, control construction, autonomous selection</td>
            </tr>
            <tr>
                <td>Robot</td>
                <td>Robot lifecycle methods and command scheduler</td>
            </tr>
        </tbody>
    </table>

    <p>Do not put button logic inside a subsystem.</p>

    <p>Do not put direct motor-control code inside <code>ControlBoard</code>.</p>

    <p>Do not put entire autonomous routines inside <code>Robot.java</code>.</p>

    <hr>

    <h1>Testing Checklist</h1>

    <p>Before testing a new subsystem:</p>

    <ul>
        <li>Verify the CAN ID.</li>
        <li>Verify motor inversion.</li>
        <li>Keep mechanisms safely elevated or disconnected when possible.</li>
        <li>Start with low voltage, low percent output, or low target speed.</li>
        <li>Confirm the desired state appears in SmartDashboard.</li>
        <li>Confirm sensor position and velocity update correctly.</li>
        <li>Confirm current draw is reasonable.</li>
        <li>Watch motor temperature.</li>
        <li>Confirm <code>IDLE</code> stops motors that should not remain powered.</li>
        <li>Tune <code>kP</code> before adjusting <code>kI</code> and <code>kD</code>.</li>
        <li>Test autonomous routines in a clear area.</li>
    </ul>

    <hr>

    <h1>Common Problems</h1>

    <table>
        <thead>
            <tr>
                <th>Problem</th>
                <th>Likely Cause</th>
                <th>First Check</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Motor does not move</td>
                <td>Wrong CAN ID, disabled robot, command not scheduled</td>
                <td>CAN ID, robot enabled state, subsystem dashboard state</td>
            </tr>
            <tr>
                <td>Motor moves backward</td>
                <td>Incorrect inversion</td>
                <td><code>InvertedValue</code> in subsystem configuration</td>
            </tr>
            <tr>
                <td>Arm targets wrong position</td>
                <td>Encoder units or setpoints are incorrect</td>
                <td>Actual position dashboard value and target position</td>
            </tr>
            <tr>
                <td>Shooter does not reach target speed</td>
                <td>Incorrect PID, friction, battery voltage, inversion</td>
                <td>Actual velocity, target velocity, current draw</td>
            </tr>
            <tr>
                <td>Shooter remains on</td>
                <td>Missing or broken <code>IDLE</code> behavior</td>
                <td>Shooter default command and <code>IDLE</code> state</td>
            </tr>
            <tr>
                <td>Two commands fight each other</td>
                <td>Commands have incorrect subsystem requirements</td>
                <td>Command construction and <code>addRequirements()</code></td>
            </tr>
            <tr>
                <td>NetworkTables update has no effect</td>
                <td>Motor configuration was applied only in constructor</td>
                <td>Add an intentional config-apply command</td>
            </tr>
            <tr>
                <td>Joystick drifts</td>
                <td>Deadband too small</td>
                <td>Driver deadband NetworkTables entry</td>
            </tr>
        </tbody>
    </table>

    <hr>

    <div class="center">
        <h2>SW Robotics Code Standard</h2>
        <p><strong>Build clearly. Log useful data. Tune carefully. Test safely.</strong></p>
    </div>

</div>

</body>
</html>