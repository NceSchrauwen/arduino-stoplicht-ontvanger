//Opdracht 9 - For receiving, processing and acting on the Bluetooth commands (via HC-05) send by an android device to be able to use a stoplight in 2 different modes.
//Written by: Nina Schrauwen 

// Include the software serial library 
#include <SoftwareSerial.h>

// Define the bluetooth device and the corresponding pins 
SoftwareSerial BTserial(10, 11); // RX | TX
static String command = "";  // Static to maintain state between calls

// Define constant variables 
const int LED_R = 2;
const int LED_O = 3;
const int LED_G = 4;

const int DEFAULT_INTERVAL = 1000;

// Define the commands given to the commands used in the logic 
const String DEFAULT_MODE = "DEFAULT";
const String STOPLIGHT_MODE = "STOPLIGHT";

// Define the state and timing variables 
unsigned long currentMillis;
unsigned long previousMillis = 0;

bool inSequenceMode = false;
enum SequenceState { RED, ORANGE, GREEN };
SequenceState currentSequenceState = RED;
unsigned long phaseStartTime;
unsigned long phaseDuration;

void setup() {
  // Define led outputs 
  pinMode(LED_R, OUTPUT);
  pinMode(LED_O, OUTPUT);
  pinMode(LED_G, OUTPUT);
  // Begin serial communication 
  Serial.begin(9600);
  // Begin the bluetooth communication
  BTserial.begin(9600);
  Serial.println("Bluetooth starting...");

  delay(2000);  // Wait 2 seconds for the HC-05 to initialize

  while (BTserial.available()) BTserial.read();  // Clear any initial garbage data
  
  // When no mode is selected, default mode should be on 
  default_mode();
}

// Function to receive the commands via bluetooth 
String readBluetoothCommand() {
  // Read the signal when there is bluetooth available
  while (BTserial.available()) {
    char receivedChar = BTserial.read();
	
	// If the received bit is a new line command then the full command has come through
    if (receivedChar == '\n') {
	  // The completed command is returned and the command variable gets emptied for the next command to come through
      String completeCommand = command;
      command = "";
      return completeCommand;
	// If no new line command is received yet add the new character to the command variable 
    } else {
      command += receivedChar;
    }
  }
  // If no command is received return an empty string 
  return "";
}

// TODO: Comment from this section on! 
// Default mode function
void default_mode(){
  // Capture the current time 
  currentMillis = millis();
  // If the interval has elapsed, previousMillis gets assigned currentMillis
  if (currentMillis - previousMillis >= DEFAULT_INTERVAL) {
    previousMillis = currentMillis;
	// Toggle the led state of the orange led (if previously it was off, turn it on / if previously it was on, turn it off)
    digitalWrite(LED_O, !digitalRead(LED_O)); 
  }
  // Ensure red and green leds are off 
  digitalWrite(LED_R, LOW);  
  digitalWrite(LED_G, LOW);  
}

// Function to handle the Bluetooth command input
void handleCommandInput(String command){
  command.toUpperCase();  // Convert command to uppercase
	
	// If the Bluetooth command is DEFAULT_MODE
	if(command == DEFAULT_MODE){
		// Will make sure default mode will be activated within the loop function
		inSequenceMode = false;
		Serial.println("Default Mode Activated via Bluetooth");
	// If the Bluetooth command is STOPLIGHT_MODE
	} else if(command == STOPLIGHT_MODE){
		// Will make sure stoplight mode will be activated within the loop function
		inSequenceMode = true;
		// Ensures smooth transition to the red light of this mode
		digitalWrite(LED_O, LOW);  
		// Initialize the sequence state
		currentSequenceState = RED; 
		// Capture the current time 
		phaseStartTime = millis(); 
		// Pass along the duration of the RED phase which is randomly chosen between 8 and 15 seconds		
		phaseDuration = random(8000, 15000); 
		Serial.println("Stoplight Mode Activated via Bluetooth");
	// Otherwise it's an unknown bluetooth command, print it to verify it's not the right command with noise added for example
	} else {
		Serial.print("Unknown Bluetooth Command: ");
        Serial.println(command);
	}
  
}

void loop() {
  // Get the Bluetooth command, if available
  String command = readBluetoothCommand();
  
  // Check if a valid command was received
  if (command != "") {
    Serial.print("Processing command: ");
    Serial.println(command);
	// Call the function handleCommandInput to process and act on the command
	handleCommandInput(command);
  }
  
  // If inSequenceMode is true run the runSequence function (STOPLIGHT_MODE)
  if(inSequenceMode){
	  runSequence();
  // If inSequenceMode is false run the default_mode function (DEFAULT_MODE)
  } else {
	  default_mode();
  }

  // Small delay for stability
  delay(100);  
  
}

// Function to execute the STOPLIGHT_MODE
void runSequence(){
   // Get current time 
   currentMillis = millis();
   
   // Switch between all 3 led states for smooth transition between modes
   switch(currentSequenceState){
	   // When the sequence state is RED turn on the red led 
	   case RED:
		digitalWrite(LED_R, HIGH);
		// Check if the phaseDuration has been elapsed  
		if(currentMillis - phaseStartTime >= phaseDuration){
			// Turn of red led 
			digitalWrite(LED_R, LOW);
			Serial.print("Red LED Duration: ");
			Serial.print(phaseDuration / 1000.0);
			Serial.println(" seconds");

			// Move to Orange phase, set the currentSequenceState to ORANGE
			// Then capture current time and pass along it's phaseDuration
			currentSequenceState = ORANGE;
			phaseStartTime = millis();
			phaseDuration = 3000;
		}
		break;
		
		// When the sequence state is ORANGE turn on the orange led 
		case ORANGE:
		digitalWrite(LED_O, HIGH);
		if(currentMillis - phaseStartTime >= phaseDuration){
			// Check if the phaseDuration has been elapsed
			digitalWrite(LED_O, LOW);
			Serial.print("Orange LED Duration: ");
			Serial.print(phaseDuration / 1000.0);
			Serial.println(" seconds");
			
			// Move to Green phase, set the currentSequenceState to GREEN
			// Then capture current time and pass along it's phaseDuration
			currentSequenceState = GREEN;
			phaseStartTime = millis();
			// phaseDuration is randomly chosen between 8 and 15 seconds 
			phaseDuration = random(8000, 15000);
		}
		break;
		
		// When the sequence state is GREEN turn on the green led 
		case GREEN:
		digitalWrite(LED_G, HIGH);
		if(currentMillis - phaseStartTime >= phaseDuration){
			// Check if the phaseDuration has been elapsed
			digitalWrite(LED_G, LOW);
			Serial.print("Green LED Duration: ");
			Serial.print(phaseDuration / 1000.0);
			Serial.println(" seconds");
			
			// Circle back to the Red phase, set the currentSequenceState to RED
			// Then capture current time and pass along it's phaseDuration
			currentSequenceState = RED;
			phaseStartTime = millis();
			// phaseDuration is randomly chosen between 8 and 15 seconds 
			phaseDuration = random(8000, 15000);
		}
		break;
	   
   }
	
}
