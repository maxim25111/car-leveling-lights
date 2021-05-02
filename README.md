# car-leveling-lights
This is 10 level adiustment for car leveling lights. Is using 7 segment display because was fitting in to small space of oryginal housing.


"
struct NUMBERS{
  bool led[10][8] = { {1,1,1,1,1,1,0,0},//0
                      {0,0,0,0,1,1,0,0},//1
                      {1,1,0,1,1,0,1,0},//2
                      {1,0,0,1,1,1,1,0},//3
                      {0,0,1,0,1,1,1,0},//4
                      {1,0,1,1,0,1,1,0},//5
                      {1,1,1,1,0,1,1,0},//6
                      {0,0,0,1,1,1,0,0},//7
                      {1,1,1,1,1,1,1,0},//8
                      {1,0,1,1,1,1,1,0}};//9
                      
  int pin[8] = {PA0, PA1, PA2, PA3, PA4,PA5, PA6, PA7};
  int button1 = PA8;
  int button2 = PA9;
  int output = PB6;// output voltage for carlight.
  int number = 0;// level of lights
  int lastNumber = 0;

  void setPinMode(){ // basic setting of pinout
    for ( int c = 0; c < 8; c++ ){
      pinMode(pin[c], OUTPUT);
    }
    pinMode(button1, INPUT_PULLUP);
    pinMode(button2, INPUT_PULLUP);
    pinMode(output, PWM);
  }

  void checkButtonState(){
    if ( digitalRead(button1) == 0){
      number++;
    }
    else if( digitalRead(button2) == 0 ){
      number--;
    }
    
    while ( digitalRead(button1) == 0 || digitalRead(button2) == 0 ){
      crasyLogo( 1 );
    }
    
    if ( number > 9 ){  // seting range and make loop
      number = 0;
    }
    if ( number < 0 ){
      number = 9;
    }
  }

  void displayAll(){  // Testing display if numbers are properly displayed
    for ( int i = 0; i < 10; i++ ){
      for ( int a = 0; a < 15; a++){
        for ( int c = 0; c < 8; c++){
          digitalWrite(pin[c], led[i][c] );
          delay(1);
          digitalWrite(pin[c], 0 );
          delay(1);
        } 
      }
    }
  }
  
  void display(){ // display actual level of lights
    cleanDisplay();
    for ( int c = 0; c < 8; c++){
      digitalWrite(pin[c], led[number][c] );
      delay(1);
      digitalWrite(pin[c], 0 );
      delay(1);
    }
  }

  void cleanDisplay(){
    for ( int c = 0; c < 8; c++){
      digitalWrite(pin[c], 0);
    }
  }

  bool numberChanged(){
    if ( number != lastNumber ){
      return true;
      lastNumber = number;
    }
    else {
      return false;
    }
  }

  void setOutput(){
    if ( numberChanged() ){
      int out = 0;
      int oneStep = 65535 / 11;
      
      out = oneStep * (number + 2);
      pwmWrite(output, out);
    }
    if ( number == 0 ){ // sequring output when level is 0. 
      pwmWrite(output, 13106); // for some reason 0 dont update and this is manual corecion
    }  // Car lights when see 0V thinks that conection is brake, and is not
       //updating level of lights. Making 11 steps, and shift number in 2 place fixt that problem.

  }
  
  void initiateLights(){
    number = 9;
    setOutput();
    
    crasyLogo( 20 );
    
    number = 0;
    setOutput();
  }

  void crasyLogo( int howMany ){
    for ( int a = 0; a < howMany; a++){
        for ( int c = 0; c < 8; c++){
          digitalWrite(pin[c], led[8][c] );
          delay(10);
          digitalWrite(pin[c], 0 );
          delay(10);
        } 
     }
   }
};

void setup() {
  // put your setup code here, to run once:
  NUMBERS numbers;
  numbers.setPinMode();
  numbers.displayAll();
  numbers.initiateLights();

  while(1){
    numbers.checkButtonState();
    numbers.display();
    numbers.setOutput();
  }
}

void loop() {
  // put your main code here, to run repeatedly:
}
"
