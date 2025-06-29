#include <reg51.h>

// Function prototypes
void ms_delay(unsigned int time);
void lcd_command(unsigned char cmd);
void lcd_data(unsigned char dat);
void lcd_print_num(unsigned int num);

// LCD control pins
sbit RS = P2^0;
sbit RW = P2^1;
sbit EN = P2^2;

// Sensors and peripherals
sbit ENTRY_IR = P3^0;   // Entry IR sensor
sbit EXIT_IR = P3^1;    // Exit IR sensor
sbit RESET_BTN = P3^2;  // Reset button
sbit BUZZ = P3^3;       // Buzzer

void main()
{
    unsigned char i;
    unsigned int count = 0;
    bit prev_entry = 1;
    bit prev_exit = 1;

    char str1[] = "Room Occupancy";
    char str2[] = "Count= ";

    // Initialization
    P1 = 0x00;
    RS=0; RW=0; EN=0;
    BUZZ=0;

    lcd_command(0x38); // 8-bit mode
    lcd_command(0x0C); // Display ON
    lcd_command(0x06); // Entry mode
    lcd_command(0x01); // Clear
    ms_delay(2);

    // Display static text
    lcd_command(0x80);
    for(i=0; str1[i]!='\0'; i++)
        lcd_data(str1[i]);

    lcd_command(0xC0);
    for(i=0; str2[i]!='\0'; i++)
        lcd_data(str2[i]);

    // Show initial count
    lcd_command(0xC6);
    lcd_print_num(count);

    while(1)
    {
        // ENTRY detection
        if(ENTRY_IR==0 && prev_entry==1)
        {
            count++;
            lcd_command(0xC6);
            lcd_print_num(count);

            BUZZ=1; ms_delay(100); BUZZ=0;

            prev_entry=0;
            ms_delay(300);
        }
        if(ENTRY_IR==1)
        {
            prev_entry=1;
        }

        // EXIT detection
        if(EXIT_IR==0 && prev_exit==1)
        {
            if(count>0) count--;
            lcd_command(0xC6);
            lcd_print_num(count);

            BUZZ=1; ms_delay(100); BUZZ=0;

            prev_exit=0;
            ms_delay(300);
        }
        if(EXIT_IR==1)
        {
            prev_exit=1;
        }

        // RESET button
        if(RESET_BTN==0)
        {
            count=0;
            lcd_command(0xC6);
            lcd_print_num(count);

            BUZZ=1; ms_delay(300); BUZZ=0;

            while(RESET_BTN==0); // wait for release
        }
    }
}

// LCD command
void lcd_command(unsigned char cmd)
{
    RS=0;
    RW=0;
    P1=cmd;
    EN=1;
    ms_delay(5);
    EN=0;
    ms_delay(5);
}

// LCD data
void lcd_data(unsigned char dat)
{
    RS=1;
    RW=0;
    P1=dat;
    EN=1;
    ms_delay(5);
    EN=0;
    ms_delay(5);
}

// Delay
void ms_delay(unsigned int time)
{
    unsigned int i,j;
    for(i=0;i<time;i++)
        for(j=0;j<113;j++);
}

// Display number
void lcd_print_num(unsigned int num)
{
    unsigned char hundreds, tens, units;
    hundreds = num / 100;
    tens = (num / 10) % 10;
    units = num % 10;

    lcd_data(hundreds+'0');
    lcd_data(tens+'0');
    lcd_data(units+'0');
}
