#pragma config(Motor,  port2,           leftWheel,      tmotorVex393_MC29, openLoop)
#pragma config(Motor,  port3,           rightWheel,     tmotorVex393_MC29, openLoop)
#pragma config(Motor,  port4,           flipperRight,   tmotorVex393_MC29, openLoop)
#pragma config(Motor,  port5,           flipperLeft,    tmotorVex393_MC29, openLoop)
#pragma config(Motor,  port6,           armLeft,        tmotorVex393_MC29, openLoop)
#pragma config(Motor,  port7,           armRight,       tmotorVex393_MC29, openLoop)
//*!!Code automatically generated by 'ROBOTC' configuration wizard               !!*//

//************************
//         CONFIG
//************************
#pragma platform(VEX2)
#pragma competitionControl(Competition)
#include "Vex_Competition_Includes.c"


//************************
//       VARIABLES
//************************
bool up = true;   // Whether the carrier is up or not
//float timeForward = 0;	// how much time it took to reach mobile goal and remove if you do not use timer

//*************************
//         METHODS
//*************************

/***** MOVEMENT *****/

/* Move the robot forward/backward
		wheelPower: [-127, 127] how fast the robot goes
				(+) => forwards
				(-) => backwards      */
void moveWheels(int wheelPower) {
	motor[leftWheel] = -wheelPower;
	motor[rightWheel] = wheelPower-1; //dont change! made so robot does not curve
}

/* Stops all wheel movement */
void stopWheels() {
	motor[leftWheel] = 0;
	motor[rightWheel] = 0;
}

/* Turns 180 counterclockwise */
void turnLeft() {
	motor[leftWheel] = -60;
	motor[rightWheel] = -60;
	sleep(2750);
	stopWheels();
}

/* Turns 180 clockwise */
void turnRight() {
	motor[leftWheel] = 60;
	motor[rightWheel] = 60;
	sleep(2750);
	stopWheels();
}


//***** FLIPPER *****
void flipCap(){
  	motor[fliperLeft] = -60;//turns the cap over... hopefully
	motor[flipperRight] = -60;
	sleep(2750);
	stopWheels();
}
   
//****** ARM *******
void moveArm(){
  	motor[armLeft] = -60;//should lift the arm
	motor[armRight] = -60;
	sleep(2750);
	stopWheels();
}
//***** AUTONOMOUS *****


//parking bonus
void auto(){
	//flip cap
	//moveArm()
	//flipCap();
	moveWheels(40);
	sleep(100);
	stopWheels();
	//moveArm();// need a new method to move down
}


//***** DRIVING *****
/* Controls:		
		left joystick = treads
		6D = arm down
		6U = arm up
		5D = cap clockwise
		5U = cap counterclockwise
*/

/* Curves the input to give a smoother driving experience
https://www.desmos.com/calculator/xv2hbpabjm        */
int modify(int input) {
	return (input+(pow(input,5)/8192-pow(input,3))/8192)/3;
}

//it needed to be faster so i made a slight alteration
//https://www.desmos.com/calculator/xv2hbpabjm
int modify2(int input) {
	return (input+(pow(input,5)/8192-pow(input,3))/8192)/2;
}

/* Update wheel powers based on controller */
void updateWheels() {
	motor[leftWheel] = -modify2(vexRT[Ch3]);
	motor[rightWheel] = modify2(vexRT[Ch3]);
}

/* Update claw power based on controller */
void updateFlipper() {
	if(vexRT[Btn5U] == 1) // If upper Z button down
		motor[flipperLeft] = 60;
		motor[flipperRight] = -60;
	else if(vexRT[Btn5D] == 1) // If lower Z button down
		motor[flipperLeft] = -60;
		motor[flipperRight] = 60;
	else
		motor[flipperLeft] = 0;
		motor[flipperRight] = 0;
}

void updateArm() {
	if(vexRT[Btn6U] == 1) // If upper Z button down
		motor[armLeft] = 60;
		motor[armRight] = -60;
	else if(vexRT[Btn6D] == 1) // If lower Z button down
		motor[armLeft] = -60;
		motor[armRight] = 60;
	else
		motor[armLeft] = 0;
		motor[armRight] = 0;
}


/* Updates carrier powers based on controller */
void updateMobileGoal() {
	// If carrier is currently up and lower button pressed
	if(up && vexRT[Btn8R] == 1) {
		lowerCarrier();
		up = false;
	// If carrier is currently down and raise button pressed
	} else if(!up && vexRT[Btn8D] == 1) {
		raiseCarrier();
		up = true;
	}
}


//*************************
//          TASKS
//*************************

void pre_auton() {
	bStopTasksBetweenModes = true;
}

task autonomous() {
	auto();
}

task usercontrol() {
	while (true) {
		updateWheels();
		updateFlipper();
		updateArm();
		updateMobileGoal();
	}
}