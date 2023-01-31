clearWatch();

I2C1.setup({scl:B6,sda:B7, bitrate:100000});
var bme = require("BME280").connect(I2C1);
var mpu = require("MPU6050").connect(I2C1);

mpu.initialize();

var Btemp, Bytemp, Bypres, Byhumid, Bhumid, Mytemp, Baltit, Byaltit, Mygrav, Myrota, Mydegr, Myacc, accCascade, MydegrCascade, MygravCascade, MyrotaCascade;
var accCas=0, MydegrCas=0, MygravCas=0, MyrotaCas=0, Bytempe=0, Bypressu=0;
var m=0, n=0, PRESSED_A=0, PRESSED_B=0, w=0, p=0, y=0, Sum=0, Average=0, AccTreshold=0, AccTresh=0;

var http = require("http");

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  res.write('<html><body><p>Result:</p><div id="result">Loading...</div></body></html>');
  res.end();
}).listen(8080);

setInterval(function() {
  var MydegrC = Math.random() * 180;
  var Bypressur = Math.random() * 10;
  var MyrotaC = Math.random() * 360;
  var MygravC = Math.random() * 10;
  var Bytemper = Math.random() * 100;
  var accC = Math.random() * 10;
  var LiftTreshold = Math.random() * 10;
  var StairTreshold = Math.random() * 10;

  bme.readRawData();
  Btemp = bme.calibration_T(bme.temp_raw);
  Bpres = bme.calibration_P(bme.pres_raw);
  Bhumid = bme.calibration_H(bme.hum_raw);
//  Baltit = bme.calibration_A(bme.alti_raw);

  Bytemp = Btemp/100.0;
  Bypres = Bpres/100.0;
  Byhumid = Bhumid/1024.0;
  Myacc = mpu.getAcceleration();
  Mytemp = mpu.getTemperature();
  Mygrav = mpu.getGravity();
  Myrota = mpu.getRotation();
  Mydegr = mpu.getDegreesPerSecond();
  var j;
  for (j=0;j>3;j++){
  	Myacc[j] = (Myacc[j]/16384) * 9.80665;
    Mygrav[j] = (Mygrav[j]/16384) * 9.80665;
    Myrota[j] = (Myrota[j]/16384) * 9.80665;
    Mydegr[j] = (Mydegr[j]/16384) * 9.80665;
  }
  accCascade = Math.sqrt(Myacc[0]*Myacc[0] + Myacc[1]*Myacc[1] + Myacc[2]*Myacc[2]);
  MygravCascade = Math.sqrt(Mygrav[0]*Mygrav[0] + Mygrav[1]*Mygrav[1] + Mygrav[2]*Mygrav[2]);
    MyrotaCascade = Math.sqrt(Myrota[0]*Myrota[0] + Myrota[1]*Myrota[1] + Myrota[2]*Myrota[2]);
    MydegrCascade = Math.sqrt(Mydegr[0]*Mydegr[0] + Mydegr[1]*Mydegr[1] + Mydegr[2]*Mydegr[2]);

  var k;
  for (k=0;k<10;k++){
  w=w+1;
  y=y+1;
  accCas = accCas + accCascade;   //Sum up acc
  MygravCas = MygravCas + MygravCascade;   //Sum up grav
  MyrotaCas = MyrotaCas + MyrotaCascade;   //Sum up rot
  MydegrCas = MydegrCas + MydegrCascade;   //Sum up deg
  Bytempe = Bytempe + Bytemp;
  Bypressu = Bypressu + Bypres;
    if (w==5){
      LiftTreshold=Bypres;
    }
    if (y==5){
      StairTreshold=Bypres;
    }
  }

accC = (accCas/10); //averaging acc
MygravC = (MygravCas/10); //averaging grav
MyrotaC = (MyrotaCas/10); //averaging rot
MydegrC = (MydegrCas/10); //averaging deg
Bytemper = (Bytempe/10); //averaging temp
Bypressur = (Bypressu/10); //averaging press

accCas=0;
MydegrCas=0;
MygravCas=0;
MyrotaCas=0;
Bytempe=0;
Bypressu=0;

//console.log("DEGREES " , "PRESSURE " , "Rotation " , "GRAVITY " ,  "Temperature " , "Acceleration ");
//console.log(" " + MydegrC, " "  + Bypressur, " "  + MyrotaC, " "  + MygravC, " "  + Bytemper, " "  + accC);
//console.log(" " + Myacc[0], " "  + Myacc[1], " "  + Myacc[2]);
//console.log(" LIFT " + LiftTreshold);
//console.log(" stairs" + StairTreshold);
//console.log(" " + Bypres);

if (MydegrC < 12) {
        n=0;
        m=m+1; // Count number of degrees treshold changes
        p=p+1;  // Count updated for Acc Treshold
        p=0; // RESET TRESHOLD AFTER STAIRS CLIMB
            if (p==3){
        AccTreshold=accC;
             }
        AccTresh= Math.abs(AccTreshold - accC);
 //   if (AccTresh < 300) {
      console.log("ValueMMM: " + LiftTreshold);
     console.log("VazzzPRESS: " + Bypres);
      if ((m > 3) && ((LiftTreshold - Bypres) == 0))
         {
          m = 0;
       //  console.log("STAIRS");
         }
         else if ((m > 5) && ((LiftTreshold+0.1) < Bypres))
         {
         console.log("GOING DOWN LIFT");
         result += "Going down lift";
         }
         else if ((m > 5) && ((LiftTreshold-0.1) > Bypres))
         {
         console.log("GOING UP LIFT");
         result += "Going up lift";
         }
         else if (m > 10)
         {
          console.log("static_AND_no motion");
          y=0; // RESET TRESHOLD AFTER STAIRS CLIMB
          m = 0;
          }
    else
    {
    // console.log("static_AND_no motion: " + m);
    result += "Static and no motion";
    }
  }

  else if (MydegrC > 12) {
    m = 0;
    n=n+1;
    p = 0;
 //   console.log("Value1: " + n);
    if ((n > 3) && ((StairTreshold - Bypres) == 0))
         {
          n = 0;
       //  console.log("STAIRS");
         }
         else if ((n > 3) && ((StairTreshold-0.05) > Bypres))
         {
         console.log("UP STAIRS");
         result += "Up stairs";
         }
         else if ((n > 3) && ((StairTreshold+0.05) < Bypres))
         {
         console.log("DOWN STAIRS");
         result += "Down stairs";
         }
         else if (n > 10)
         {
          w=0; // RESET TRESHOLD AFTER STAIRS CLIMB
          console.log("no motion");
          result += "Static and no motion";
          n = 0;
          }
    }
  else {
        console.log("NO_IDEA");
        result += "No idea";
    }

  // Update the result on the HTML page
  Espruino.Core.Serial.write("\x1b[s\x1b[0;0H" + result + "\x1b[u");
}, 200);
//end start

//Wait for program to load
setTimeout(function () {
 start();
}, 1000);