# DATA BASE PROJECT

## How to Execute

1. **Prerequisites:**
- Python 3.0 or newer
- Data Base PgAdmin installed 
   

2. **Download the Code:**
- 


3. **Run the interface.zip Script:**
- The interface.zip main.py must be run on the PC connected to the portable camera.

- Its execution must be started before robot.zip's main.py (so that socket communication takes place correctly).



4. **Run the robot.zip Script:**
- Robot.zip's main.py must be run on the PC terminal connected to the robots and the PS4 controller.

- The user must ensure that:
- Adjust the SERVER_IP in the CONSTANTS section. To check the correct value, simply use the 'ipconfig' command on the PC connected to the portable camera.
- Make sure that the COM ports associated with each robot are well defined (GLOBAL VARIABLES section).


 Note: Both scripts have usage 'messages' that are available in RUNTIME.
