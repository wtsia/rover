```

import java.util.Scanner; 

public class LabProgram {
   public static void main(String[] args) {
      Scanner scnr = new Scanner(System.in); 
      int highwayNumber = 0;
      int primaryNumber = 0;
      String highwayNumStr = "";
      String direction = "";
       char lastStrChar;
       char secondLastStrChar;
       

      highwayNumber = scnr.nextInt();
      primaryNumber = highwayNumber;
      String notValid = highwayNumber + " is not a valid interstate highway number.";
      
      /* Type your code here. */
      highwayNumStr = highwayNumStr.valueOf(highwayNumber);
       
       
       if (highwayNumber%2 == 0) {
         direction = "east/west.";
      } else {
         direction = "north/south.";
      }
                
      if (highwayNumber > 0) {
         if (highwayNumber < 100) {
            System.out.println("I-" + highwayNumber + " is primary, going " + direction);
         } else {
            lastStrChar = highwayNumStr.charAt(highwayNumStr.length()-1);
             secondLastStrChar = highwayNumStr.charAt(highwayNumStr.length()-2);
            if (highwayNumber < 1000) {
               if ((lastStrChar == '0') & (secondLastStrChar == '0')) {
                  System.out.println(notValid);
               } else if (secondLastStrChar == '0') {
                  System.out.println("I-" + highwayNumber + " is auxiliary, serving I-" + lastStrChar + ", going " + direction);
               } else {
                  System.out.println("I-" + highwayNumber + " is auxiliary, serving I-" + secondLastStrChar + lastStrChar + ", going " + direction);
               }
            } else {
               System.out.println(notValid);
            }
         }
      } else {
         System.out.println(notValid);
      }
   }
}

```