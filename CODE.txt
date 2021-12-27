/*
    * File:   main.c
 * Author: Asaad Dadoush
 *
 * Created on November 26, 2021, 1:27 AM
 */


#include <xc.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include "config.h"
#include "lcd.h"

void setup(void);
void loading(void);

#define idle "Loading"

int access_count = 0;
char myStg[10];
int enter_count = 0;
char myStr[10];
int timer;
char mycount[10];


void lcd_info(){
    
    Lcd_Set_Cursor(1,1);
    Lcd_Write_String("Enter: ");
    Lcd_Set_Cursor(1,7);
    sprintf(myStr, "%d", enter_count);  
    Lcd_Write_String(myStr);
        
        
    Lcd_Set_Cursor(2,1);
    Lcd_Write_String("Access: ");
    Lcd_Set_Cursor(2,8);
    sprintf(myStg, "%d", access_count);  
    Lcd_Write_String(myStg);
    __delay_ms(500);
    
}


void count(){
    enter_count++;
    Lcd_Set_Cursor(1,7);
    sprintf(myStr, "%d", enter_count);  
    Lcd_Write_String(myStr);
    return;
}

void timerdown(){
    
    while(1){
       if(INT1IF== 1){
           break;
       }
       Lcd_Set_Cursor(1,9);
       Lcd_Write_String(" Counter");
       __delay_ms(1000);
       Lcd_Set_Cursor(2,12);
       sprintf(mycount, "%d", timer);
       Lcd_Write_String(mycount);
       if(timer == 9){
           Lcd_Set_Cursor(2,13);
           Lcd_Write_String(" ");
       }
        --timer;
        if(timer == 0){
            break;
        }
    }
    if(INT1IF==0){
        __delay_ms(1000);
        Lcd_Set_Cursor(1,9);
        Lcd_Write_String("        ");
        Lcd_Set_Cursor(2,12);
        Lcd_Write_String("   ");
    }
    return;   
}

void alarm(){
    Lcd_Clear();
    while(INT1IF == 0){
        __delay_ms(300);
        LATC2 = !LATC2;
        LATC5 =  !LATC5; 
        Lcd_Set_Cursor(1,1);
        Lcd_Write_String("Please sanitize");
        Lcd_Set_Cursor(2,1);
        Lcd_Write_String("   your hands");

    }
    return;
}

void access(){
        LATC2 = 0; // RED
        Lcd_Clear();
        Lcd_Set_Cursor(1,1);
        Lcd_Write_String("======Done======");
        unsigned int L = 4;
        while(L > 0){
            __delay_ms(80);
            LATC3 = !LATC3; // GREEN
            LATC5 = !LATC5; // BUZ
            L--;
        }
        if(access_count < enter_count){
            access_count++;
        }
        LATC3 = 0; // GREEN
        LATC5 = 0;
      
        return;
}

void __interrupt() intrr(){
    if(INT0IF && INT0IE){
        timer = 10;
        count();
        timerdown();
        if(LATB1 == 0){
            alarm();
            }
        INT0IF = 0;

    }
    if(INT1IF && INT1IE){
        access();
        __delay_ms(500);
        Lcd_Clear();
        lcd_info();
        
        INT1IF =0;
    }
    return;
}

void main(void) {
    loading();
    setup();
    Lcd_Clear();
    while(1){
     
      lcd_info();
       
    }
            
    return;
}

void setup(void){
    LATC = 0;
    ANCON0 = 0;     //only digital pins
    ANCON1 = 0;
    OSCCON = 0x60;  // 8 MHz
    // initialize  ports I/O
    
    TRISC = 0;
    TRISB0 = 1;
    TRISB1 = 1;
    // enable interrupts
    INT0IF = 0;
    INT0IE = 1;
    INT1IF = 0;
    INT1IE = 1;
    GIE = 1;
    
}
  // LCD configuration
void loading(){
   TRISD = 0;
   TRISE = 0;
   Lcd_Init(); 
   Lcd_Set_Cursor(1,1);
   Lcd_Write_String(idle); 
   unsigned i = 5;
   while(i > 0){
    Lcd_Write_String(".");
    __delay_ms(500);
    i--;
   }
   Lcd_Clear();
}


