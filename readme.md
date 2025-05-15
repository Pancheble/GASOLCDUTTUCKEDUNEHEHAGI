# ~ 가소 LCD 어떻게든 이해하기 ~
# USER CODE BEGIN Includes
``` C
/* USER CODE BEGIN Includes */
#include <stdio.h>
/* USER CODE END Includes */
```
<br>input output을 해야하기 때문에
<br>standard input output (stdio.h)을 해준다.

# USER CODE BEGIN PD
``` C
/* USER CODE BEGIN PD */
#define SLAVE_ADDRESS_LCD 0x4e
/* USER CODE END PD */
```
<br>0x4e는 LCD의 고유 주소이다.
<br>0x4e라고 쓰면 뭔지 모르니까, SLAVE_ADDRESS_LCD라는 변수를 써준다.

# USER CODE BEGIN 0
``` C
/* USER CODE BEGIN 0 */
void lcd_send_cmd(char cmd);
void lcd_send_data(char data);
void lcd_init();
void lcd_clear();
void lcd_put_cur(int col, int row);
void lcd_send_string(char *str);
/* USER CODE END 0 */
```
<br>만든 함수를 여기에서 정의를 해준다.
<br>보통 여기에서 정의를 한다.

# USER CODE BEGIN WHILE
``` C
/* USER CODE BEGIN WHILE */
while (1){
    char msg[16];
    lcd_put_cur(0, 0); // 1행 1열
    lcd_send_string("Incheon Elec");
    lcd_put_cur(1, 0); // 2행 1열
    sprintf(msg, "Current Val:%2d", count);
    lcd_send_string(msg);
    HAL_Delay(1000);
    count ++;
    if(count > 99){
        count = 0;
    }
  /* USER CODE END WHILE */
}
```
<br>실제로 코드가 돌아가는 곳이다.
<br>코드가 좀 많으니까 나눠서 보도록 하자.

### 1행 LCD
``` C
lcd_put_cur(0, 0); // 1행 1열
lcd_send_string("Incheon Elec");
```
<br>첫번째 줄은 1행 1열로 커서를 옮기는 코드이다.
<br>두번째 줄은 커서에서 Incheon Elec라는 문자열을 LCD에 출력하는는 코드이다.

### 2행 LCD
``` C
char msg[16];
lcd_put_cur(1, 0); // 2행 1열
sprintf(msg, "Current Val:%2d", count);
lcd_send_string(msg);
HAL_Delay(1000);
count ++;
if(count > 99){
    count = 0;
}
```
<br>뭔가 많아보이는데 일단 쫄지 말자.
``` C
char msg[16];
```
<br>이 코드는 2행 1열에 쓸 문자를 저장할 자리를 만들 코드다.
<br>행이 16개니까 그냥 16쓴거다. 다른의미는 없다.
``` C
lcd_put_cur(1, 0); // 2행 1열
sprintf(msg, "Current Val:%2d", count);
lcd_send_string(msg);
```
<br>첫번째 줄은 2행 1열로 커서를 옮기는 코드이다.
<br>두번째 줄은 smg에 문자열을 넣어주는 코드이다.
<br>count가 %2d에 들어가서 smg에 저장을 해주겠다. 라는 뜻이다.
<br>세번째 줄은 문자열이 들어간 smg를 LCD에 출력하는거다.
``` C
HAL_Delay(1000);
count ++;
if(count > 99){
    count = 0;
}
```
<br>이거는 가소시간에 잔거 아닌이상 이해를 쉽게 할 수 있을거다.
<br>첫번째 줄은 1초를 기다린다는 뜻이다.
<br>두번째 줄은 count에 1을 올리겠다는 뜻이다.
<br>세번째와 네번째 줄은 count가 99를 초과이면 count를 0으로 바꾸겠다. 라는 코드이다.

# USER CODE BEGIN 4

``` C
/* USER CODE BEGIN 4 */
void lcd_send_cmd(char cmd){
	char data_u, data_l;
	uint8_t data_t[4];
	data_u = cmd & 0xf0;;
	data_l = (cmd << 4) & 0xf0;

	data_t[0]=data_u|0x0c;
	data_t[1]=data_u|0x08;
	data_t[2]=data_l|0x0c;
	data_t[3]=data_l|0x08;

	HAL_I2C_Master_Transmit(&hi2c1, SLAVE_ADDRESS_LCD, (uint8_t *)data_t, 4, 100);
}

void lcd_send_data(char data){
	char data_u, data_l;
	uint8_t data_t[4];
	data_u = data & 0xf0;;
	data_l = (data << 4) & 0xf0;

	data_t[0]=data_u|0x0d;
	data_t[1]=data_u|0x09;
	data_t[2]=data_l|0x0d;
	data_t[3]=data_l|0x09;

	HAL_I2C_Master_Transmit(&hi2c1, SLAVE_ADDRESS_LCD, (uint8_t *)data_t, 4, 100);
}

void lcd_init(){
    HAL_Delay(50);
	lcd_send_cmd(0x30);
	HAL_Delay(5);
	lcd_send_cmd(0x30);
	HAL_Delay(1);
	lcd_send_cmd(0x30);


	HAL_Delay(10);
	lcd_send_cmd(0x20);
	HAL_Delay(10);
	lcd_send_cmd(0x28);
	HAL_Delay(1);
	lcd_send_cmd(0x08);
	HAL_Delay(1);
	lcd_send_cmd(0x01);
	HAL_Delay(1);
	lcd_send_cmd(0x06);
	HAL_Delay(1);
	lcd_send_cmd(0x0c);
}

void lcd_clear(){
	lcd_send_cmd(0x80);
	for(int i=0; i<16; i++){
		lcd_send_data(' ');
	}
	lcd_send_cmd(0xc0);
	for(int i=0; i<16; i++){
		lcd_send_data(' ');
	}
}

void lcd_put_cur(int col, int row){
	switch(col){
	case 0:
		row|=0x80;
		break;
	case 1:
		row|=0xc0;
		break;
	}
	lcd_send_cmd(row);
}

void lcd_send_string(char *str){
	while(*str){
		lcd_send_data(*str++);
	}
}
/* USER CODE END 4 */

```

<br>이건 쫄아야 정상이다. 코드 하나하나씩 보도록 하자.

## lcd_send_cmd

``` C
void lcd_send_cmd(char cmd){
	char data_u, data_l;
	uint8_t data_t[4];
	data_u = cmd & 0xf0;;
	data_l = (cmd << 4) & 0xf0;

	data_t[0]=data_u|0x0c;
	data_t[1]=data_u|0x08;
	data_t[2]=data_l|0x0c;
	data_t[3]=data_l|0x08;

	HAL_I2C_Master_Transmit(&hi2c1, SLAVE_ADDRESS_LCD, (uint8_t *)data_t, 4, 100);
}
```

<br>먼저 이 함수에서는 제어 신호를 다룬다.
<br>```N/A E RW RS``` 이렇게 4개의 신호가 들어간다.
<br>여기에서 N/A는 1로 고정을하고
<br>E는 H->L 로 바꿔줘야한다.
<br>RW는 write이며
<br>RS는 0은 명령어고, 1을 데이터이다.

``` C
char data_u, data_l;
uint8_t data_t[4];
data_u = cmd & 0xf0;
data_l = (cmd << 4) & 0xf0;
```

<br>upper: ```D7 D6 D5 D4```, lower: ```D3 D2 D1 D0```
<br>cmd값은 이렇게 생겼다.
<br>우리가 여기에서 사용해야하는 값은 제어신호가 들어갈 lower이기 때문에
<br>lower 값을 다 ```0 0 0 0```으로 바꿔주는 행위를 해야한다.

``` C
data_u = cmd & 0xf0;
```

<br>ㅇㅇ 맞다 이건 lower을 다 ```0 0 0 0```으로 바꿔주는 코드다.

``` C
data_l = (cmd << 4) & 0xf0;
```

<br>이 코드는 lower에 있는걸 upper로 올려주는거다.
<br>그리고 lower을 ```0 0 0 0```으로 채운다.
<br> 그럼 뭐 ```D3 D2 D1 D0 X X X X``` 이런식으로 생겼을 거 같다.

``` C
data_t[0]=data_u|0x0c; // 1 1 0 0
data_t[1]=data_u|0x08; // 1 0 0 0
data_t[2]=data_l|0x0c; // 1 1 0 0
data_t[3]=data_l|0x08; // 1 0 0 0

HAL_I2C_Master_Transmit(&hi2c1, SLAVE_ADDRESS_LCD, (uint8_t *)data_t, 4, 100);
```

<br>이건 제어 신호를 보내주는거다. 어려울거 없다.
<br>```N/A E RW RS```값을 그냥 잘 넣어주는 코드이다.

``` C
HAL_I2C_Master_Transmit(&hi2c1, SLAVE_ADDRESS_LCD, (uint8_t *)data_t, 4, 100);
```
<br> 이건 답없다 그냥 외워라.

## lcd_send_data

``` C
void lcd_send_data(char data){
	char data_u, data_l;
	uint8_t data_t[4];
	data_u = data & 0xf0;;
	data_l = (data << 4) & 0xf0;

	data_t[0]=data_u|0x0d; // 1 1 0 1
	data_t[1]=data_u|0x09; // 1 0 0 1
	data_t[2]=data_l|0x0d; // 1 1 0 1
	data_t[3]=data_l|0x09; // 1 0 0 1

	HAL_I2C_Master_Transmit(&hi2c1, SLAVE_ADDRESS_LCD, (uint8_t *)data_t, 4, 100);
}
```

<br>이건 lower값만 바뀐거다.
<br>데이터를 보내는 코드이기 때문에
<br>```RS```에 1이 들어간다.

## lcd_init

``` C
void lcd_init(){
	HAL_Delay(50);
	lcd_send_cmd(0x30);
	HAL_Delay(5);
	lcd_send_cmd(0x30);
	HAL_Delay(1);
	lcd_send_cmd(0x30);


	HAL_Delay(10);
	lcd_send_cmd(0x20);
	HAL_Delay(10);
	lcd_send_cmd(0x28);
	HAL_Delay(1);
	lcd_send_cmd(0x08);
	HAL_Delay(1);
	lcd_send_cmd(0x01);
	HAL_Delay(1);
	lcd_send_cmd(0x06);
	HAL_Delay(1);
	lcd_send_cmd(0x0c);
}
```

<br>그냥 LCD 기본 셋팅이다.

## lcd_clear

``` C
void lcd_clear(){
	lcd_send_cmd(0x80);
	for(int i=0; i<16; i++){
		lcd_send_data(' ');
	}
	lcd_send_cmd(0xc0);
	for(int i=0; i<16; i++){
		lcd_send_data(' ');
	}
}
```

``` C
lcd_send_cmd(0x80);
```
<br>```0x80```는 첫번째 줄 주소이다.

``` C
lcd_send_cmd(0xc0);
```

<br>```0xc0```는 두번째 줄 주소이다.
<br>그러고 for문 에서 한번씩 순회하며 ' '(공백) 으로 채우는것이다.

## lcd_put_cur

``` C
void lcd_put_cur(int col, int row){
	switch(col){
		case 0:
			row|=0x80;
			break;
		case 1:
			row|=0xc0;
			break;
	}
	lcd_send_cmd(row);
}
```
<br>코드에서 row(열)랑 col(행)의 뜻을 바꿨다 햇갈리지 말자.
<br>switch문으로 col(열)은 ```0 1``` 만 들어갈 수 있다.

``` C
switch(col){
	case 0:
		row|=0x80; // 첫번째 열
		break;
	case 1:
		row|=0xc0; // 두번째 열
		break;
}
```

<br> 그리고 행 값을 넘겨준다

``` C
lcd_send_cmd(row);
```

## lcd_send_string

``` C
void lcd_send_string(char *str){
	while(*str){
		lcd_send_data(*str++);
	}
}
```

<br> *(애스터리스크)는 포인터다 그냥 메모리에 str값을 넣겠다 라는 뜻임.
<br> str을 반복하는데 항상 str의 마지막값은 Null(False)이다.
<br> 그러면서 str을 하나하나씩 메모리 주소에 넣는다.

# 끝

<br>나도 완전하게 이해를 못해서 오류가 있을 수 있다.
<br>오류 있으면 말해라.
