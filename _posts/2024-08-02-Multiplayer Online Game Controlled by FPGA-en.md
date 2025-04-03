---
layout:     post
title:      Multiplayer Online Game Controlled by FPGA
subtitle:   2023 School Project
date:       2023-03-02
author:     JY
header-img: img/post-bg-debug.jpg
catalog: true
lang: en
tags:
    - C#
    - Unity
    - Python
    - AWS
    - Project
---

# Information Processing Report

## Purpose of the System

Our aim was to create an engaging multiplayer game using FPGAs as direction controllers. By leveraging AWS servers, local FPGA processing, and Unity infrastructure, the team developed a cooperative 2D competitive game.

## Game Description

We developed a co-op real-time multiplayer game on Unity using C#. 
- The game supports two players in real-time on the same laptop, and is highly scalable to accommodate more players with additional control nodes. 
- The objective is to fight against a randomly spawned monster. Players must work together to defeat the monster, and the game finish if any player dies. 

We aim to replicate the console multiplayer gaming experience, similar to playing a multiplayer game on one PlayStation.

## Performance Metrics

### Low Latency

We used the UDP communication protocol and a low sample rate in data transmission to reduce latency. The delay between an FPGA command and its implementation in the game is about 300ms, enhancing the user experience.

### High Accuracy

We added an FIR filter to reduce noise and set an optimal range for `data_x` and `data_y` to decide the moving direction.

### Multiplayer Cooperative Game

Using the AWS server, we transmit information to the Unity Computer to display both players’ real-time control on the same screen, mimicking the experience of a PS5 or Nintendo Switch.

### High Flexibility in Moving Direction

Players can control their game character in the X, Y, and diagonal directions.

## Design Decisions

We split our project into three stages:

1. FPGA design and data processing
2. Server-side design
3. Unity game development

We allocated team members to each stage based on their specialized knowledge to work simultaneously.

## FPGA Design and Data Processing

### Using FIR-Filtered Data

```c
#define N 49 
float COEFF[N] = {0.0046,0.0074,-0.0024,-0.0071,0.0033,0.0001,-0.0094,0.0040,0.0044,-0.0133,0.0030, 
0.0114,-0.0179,-0.0011,0.0223,-0.0225,-0.0109,0.0396,-0.0263,-0.0338,0.0752,-0.0289,-0.1204,0.2879, 
0.6369,0.2879,-0.1204,-0.0289,0.0752,-0.0338,-0.0263,0.0396,-0.0109,-0.0225,0.0223,-0.0011,-0.0179, 
0.0114,0.0030,-0.0133,0.0044,0.0040,-0.0094,0.0001,0.0033,-0.0071,-0.0024,0.0074,0.0046}; 

#define OFFSET -32 
#define PWM_PERIOD 16 
#define QUITLETTER '~' 
#define CHARLIM 64 

alt_32 firFilter(alt_32 *sample, int n, alt_32 input){ 
 int index = n-1; 
 alt_32 results; 
 results = 0; 
 for (int j = 0; j<index; j++){ 
   sample[j] = sample[j+1]; 
  } 
 for (int i= 0; i<index; i++){ 
  alt_32 num = sample[index-i]; 
  results = results + num/n; 
 } 

 results = results + input/n; 
 sample[n-1] = input; 
 return results; 
}
```

We used FIR-filtered data rather than raw data to reduce errors. MATLAB produces the 49 coefficients. The function takes an array of samples, the number of samples (\`n\`), and a new input value. It returns the filtered output value. Despite adding filters increasing latency, the rest of the system's latency optimizations keep it low, so we retained the filter for its noise-reduction performance.

### Implementing the Seven-Segment Display

```c
while(loop){ 
    printf("<--> Please Type Your Username (6 bits maximum) and Press Enter: <-->\n"); 

    char text[2*CHARLIM] = "";  
    char prevLetter = '!'; 
    int length = 0; 
    int running = 1; 
    prevLetter = alt_getchar();   
    prevLetter = get_text(prevLetter, &length, text, &running); 

    IOWR_ALTERA_AVALON_PIO_DATA(HEX5_BASE, getBin(text[0])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX4_BASE, getBin(text[1])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX3_BASE, getBin(text[2])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX2_BASE, getBin(text[3])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX1_BASE, getBin(text[4])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX0_BASE, getBin(text[5]));

    // print score 
    char text[2*CHARLIM] = ""; 
    char prevLetter = '!'; 
    int length = 0; 
    int running = 1; 
    prevLetter = alt_getchar(); 
    prevLetter = get_text(prevLetter, &length, text, &running); 

    IOWR_ALTERA_AVALON_PIO_DATA(HEX5_BASE, getBin(text[0])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX4_BASE, getBin(text[1])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX3_BASE, getBin(text[2])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX2_BASE, getBin(text[3])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX1_BASE, getBin(text[4])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX0_BASE, getBin(text[5]));
```

We designed a seven-segment display that can show both characters and numbers. It is used in two parts of the game:

1. Displaying the username when the game starts.
2. Displaying the score when the game ends.

### Implementing the Button

```c
button = IORD_ALTERA_AVALON_PIO_DATA(BUTTON_BASE); 
if(button == 0b10){//press key0 to stop 
 char stop='a'; 

    still_in_game = 0; // if will finish the game and print the mark 
    alt_up_accelerometer_spi_read_x_axis(acc_dev, & x_read); 
    alt_up_accelerometer_spi_read_y_axis(acc_dev, & y_read); 
    alt_up_accelerometer_spi_read_z_axis(acc_dev, & z_read); 
    x_read = firFilter(& samples_x, n, x_read); 
    y_read = firFilter(& samples_y, n, y_read); 
    z_read = firFilter(& samples_z, n, z_read); 
    char x_dir, y_dir; 
    x_dir = 'a'; 
    y_dir = 'a'; 
    alt_printf("%c%c\n", x_dir, y_dir); 
}
```

Pressing key 0 stops the game and displays the score on the screen.

### Data Transmission between Nios and Python

Motion data is collected and transmitted to Python for client-side processing. To address data transmission bottlenecks, we added a "usleep" function to create transmission time intervals, significantly improving performance and stability without compromising data.

The command is executed as a bash call in an internally created bash system call using the Python subprocess library. Once the button is pressed, a string “aa” is sent from the Nios, and after receiving it, the running subprocess terminates and another opens in the \`send_on_jtag1(cmd)\` function to receive the returned data of the score from the database, which is then printed on the FPGA.

## Server and Database

### Server

The server is set up on an AWS EC2 instance with Ubuntu 20.04. We chose AWS EC2 for its scalable and reliable infrastructure. Necessary software, including Apache web server, Python 3.8, and UMP, is installed. AWS CLI allows local database access.

#### Communication Protocol

We tried several communication protocols:

1. Using DynamoDB had significant delay.
2. TCP reduced delay but still had 2-3 seconds delay.
3. UDP reduced delay to below 1 second.

We chose UDP for its low latency, essential for gaming.

#### More about TCP and UDP

TCP's reliability and connection-oriented properties add overhead, causing delays. UDP, being connectionless, transmits packets faster with lower latency, making it suitable for real-time applications like gaming. However, UDP's lack of reliability can result in packet loss, leading to errors or glitches. Techniques like packet loss detection, error correction, and congestion control may be needed to improve performance.

### Data

Our game is 2D, and the attack is automatic, so we only need to transmit players' movement information. The FPGAs process and output data as two numbers representing vertical and horizontal velocities.

#### Python Code

##### Connection

```python
def init():  # connect and remember three clients 
    print("initializing, please connect player1") 
    msg1, add1 = server_socket.recvfrom(2048) 
    time.sleep(1) 

    print("initializing, please connect player2") 
    msg2, add2 = server_socket.recvfrom(2048) 
    time.sleep(1) 

    print("initializing, please connect destination") 
    msg3, add3 = server_socket.recvfrom(2048) 
    time.sleep(1) 

    return add1, add2, add3 

def connect():  # refuse and retry the connection when one player appears multiple times  
    add1, add2, add3 = init() 
    if (add1 == add2): 
            print("two player have the same address") 
            add1, add2, add3 = init() 
    if (add1 == add3 or add2 == add3): 
            print("destination and player have the same address") 
            add1, add2, add3 = init() 
    return add1, add2, add3 

add1, add2, add3 = connect()
```

This part of the code does the connection. Since we can’t know the public IP of clients, the server receives one message from each one of them and remembers the IP. If one IP appears multiple times, the server will refuse and retry the connection.

##### Transmitting

```python
while True: 
    #cadd below is the client process address 
    cmsg, cadd = server_socket.recvfrom(2048) 
    cmsg = cmsg.decode() 
    id = cmsg[:2]  # get first 3 characters of string

    # Information like scores and Health is transmitted using Database, so the transmission is one-way. We add a header in front of the message so that the game can distinguish between player1 and player2.

    if (cadd == add1): 
        cmsg = 'C1:' + cmsg 
        print("C1:" + cmsg) 
        server_socket.sendto(cmsg.encode(), add3) 

    if (cadd == add2): 
        cmsg = 'C2:' + cmsg 
        print("C2:" + cmsg) 
        server_socket.sendto(cmsg.encode(), add3)
```

### Database

The game database is designed to store various game-related data for multiple users. The database is hosted on Amazon Web Services (AWS) DynamoDB, a NoSQL database service. 

Advantages of DynamoDB:

1. Handles different types of data regardless of structure, creating flexibility in an in-game server.
2. Low latency, ideal for our real-time multiplayer game server requiring fast response times.

The functionality of the game database includes:

1. **Retrieving player data:** The \`get_player_data\` function retrieves data for a given player ID.
2. **Adding new players and their login data:** The \`new_player_data\` and \`new_login_data\` functions add new player data and login credentials, respectively.
3. **Updating player data:** The \`update_player_data\` function updates the player data in the database.
4. **Scanning tables:** The \`show_table\` function outputs all items of a specified table.

#### Test for Latency

Latency consists of software and hardware parts.

**Software Part:**

```text
Protocol/delay  Test 1    Test 2    Test 3    Test 4    Test 5    Average
UDP             165ms     343.2ms   263ms     302.3ms   187.2ms   252.14ms
```

**Hardware Part:**

```text
Running 50 times:

Average  Max       Min
0.62 ms  250.23 ms 0.53 ms
```

### Unity Game Development

To control the players' movement, the FPGA output is parsed into two values and transmitted by UDP. Data collected from the UDP server is written in two txt files to control the player's movement. After the game, the score is written in another txt file, updated in the database, and displayed on the FPGA.

Testing flow involved unit tests for each component and iterative improvements while integrating them into the final system.

## Link

To view the final system deliverable and demonstration, please refer to the video we have submitted with this coursework. It showcases live gameplay, along with commentary on the implementation. Group14

[related code](http://host.py/)

[recorded demo](https://imperiallondon-my.sharepoint.com/:v:/g/personal/bz1521_ic_ac_uk/EQWuP7PvxvNBj7r5vfuSiUABnPlbUIslF7B5U7uDMGvb-w)
