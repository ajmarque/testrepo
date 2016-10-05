//testrepo
//An attempt at creating a repository

/* ------------------------------------------------------------ */
/*				          PIC32 Timer2 Interrupt                      */
/*                 Author: Andrew J. Marques                    */
/* ------------------------------------------------------------ */

#include <p32xxxx.h>
#include <sys/attribs.h>  //must include to use interrupt macors
#include <plib.h>
#include <xc.h>
/* ------------------------------------------------------------ */
/*				PIC32 Configuration Settings		*/
/* ------------------------------------------------------------ */

/* Oscillator Settings
*/
#pragma config FNOSC		= PRIPLL	// Oscillator selection
#pragma config POSCMOD		= XT		// Primary oscillator mode
#pragma config FPLLIDIV 	= DIV_2		// PLL input divider
#pragma config FPLLMUL		= MUL_20	// PLL multiplier
#pragma config FPLLODIV 	= DIV_1		// PLL output divider
#pragma config FPBDIV		= DIV_4		// Peripheral bus clock divider
#pragma config FSOSCEN		= OFF		// Secondary oscillator enable

/* Clock control settings
*/
#pragma config IESO		= OFF		// Internal/external clock switchover
#pragma config FCKSM		= CSDCMD	// Clock switching (CSx)/Clock monitor (CMx)
#pragma config OSCIOFNC		= OFF		// Clock output on OSCO pin enable

/* USB Settings
*/
//#pragma config UPLLEN		= OFF		// USB PLL enable
//#pragma config UPLLIDIV	= DIV_2		// USB PLL input divider

/* Other Peripheral Device settings
*/
#pragma config FWDTEN		= OFF		// Watchdog timer enable
#pragma config WDTPS		= PS1024	// Watchdog timer postscaler

/* Code Protection settings
*/
#pragma config CP		= OFF		// Code protection
#pragma config BWP		= OFF		// Boot flash write protect
#pragma config PWP		= OFF		// Program flash write protect
//#pragma config JTAGEN   = OFF


/*Before using the interrupt it must be set up.  An interrupt can be triggered by 
 an external or internal source.  Each specific source coincides with a certain 
 interrupt register.  The timer 2 interrupt is associated with the 9th bit of IEC0
 register. Priority bits of each interrupt are also separate.  The priority bits 
 of timer 2 interrupt are located IPC2 <4:2>.  The same goes for sub-priority bits,
 the sub-priority bits of timer 2 are located IPC2<1:0>.  When configuring interrupts
 whole registers cannot be manipulated since this will be affecting other interrupts.*/
void InterruptT2Config(){
    //IEC0bits.T2IE = 0; //Disable timer2 interrupt my way
    IEC0CLR = 0x0200; // disable timer 2 interrupt while configuring, IEC0CLR clears specific bits in register IEC0 (interupt enabe control register 0) in this case IEC0CLR = 0b0000001000000000 which will clear bit 9 of IEC0 effectively disabling the timer 2 interupt
    IFS0bits.T2IF = 0; //Clear timer 2 interrupt flag, 1 = interrupt has been requested 0 no interrupt has been requested
    IPC2CLR = 0x001f;         // clear priority/sub-priority fields.  IPC2CLR is a clear register that works like any other.  Bits marked high will clear the corresponding bits in the name register, this case IPC2
    IPC2bits.T2IP = 4; //Set T2 interrupt priority to 4
    IPC2bits.T2IS = 0; //Set T2 interrupt sub-priority to 0
    //IPC2SET = 0x0010;         // set timer 2 int priority = 4, IPC2<4:2>
    //IPC2SET = 0x0000;         // set timer 2 int sub-priority = 0, IPC2<1:0>
    IEC0bits.T2IE = 1; //Configuration is complete, enable timer2 interrupt
}


void __ISR(8, ipl4) T2interrupt(void){
    
    LATE = 0x81; //Toggle all LED's
    IFS0bits.T2IF = 0;        // clear timer 2 int flag, IFS0<9>
}

/* ------------------------------------------------------------ */
/*			Function MAIN							*/
/* ------------------------------------------------------------ */
/*Current Peripheral Bus frequency with settings above = 20 MHz */

int main(void){

    TRISE = 0x00;
    
    T2CONSET = 0x8000; // Sets bits into T2CON.  bit 15 = 1 "ON/OFF", bits 6:4 = 111 
    PR2 = 400;
    
    INTEnableSystemSingleVectoredInt();
    InterruptT2Config();
  
    while(1){}
    
}
