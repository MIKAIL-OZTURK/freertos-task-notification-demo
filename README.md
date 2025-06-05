# FreeRTOS-Task Notification
This project was developed to illustrate the basic usage of the FreeRTOS Task Notification mechanism.  For inter-task communication, xTaskNotify() and xTaskNotifyWait() functions are used to toggle (blink) LEDs at certain intervals.

## 🔧 Project Summary
- **Hardware**: STM32F407VG Discovery Board
- **IDE**: STM32CubeIDE v1.16.1
- **RTOS**: FreeRTOS (Classic API)
- FreeRTOS Task Notification mechanism (xTaskNotify() - xTaskNotifyWait())

## Uygulama Videosu


https://github.com/user-attachments/assets/802cb512-ce88-43e5-b54c-2567ed12d697


## 4. Code
```c
#include "FreeRTOS.h"
#include "task.h"
#include "main.h"

TaskHandle_t xTaskReceiver = NULL;

#define TASK_PRIORITY_RECEIVER		(tskIDLE_PRIORITY + 2)
#define TASK_PRIORITY_SENDER		(tskIDLE_PRIORITY + 1)
#define TASK_DELAY_MS			1000


static void LedHandlerTask(void* pvParameters);
static void LedToggleSenderTask(void* pvParameters);
static void ToggleLedByNotification(uint32_t receivedValue);

int main()
{
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
  
  xTaskCreate(LedHandlerTask, "Receiver", 128, NULL, TASK_PRIORITY_RECEIVER, &xTaskReceiver);
  xTaskCreate(LedToggleSenderTask, "Sender", 128, NULL, TASK_PRIORITY_SENDER, NULL);
  vTaskStartScheduler();
  
  while(1) 
  {
  }
}

static void LedHandlerTask(void* pvParameters)
{
	uint32_t notificationValue = 0;

	for(;;)
	{
		if(xTaskNotifyWait(0, 0, &notificationValue, portMAX_DELAY) == pdTRUE)
		{
			ToggleLedByNotification(notificationValue);
		}
	}
}

static void ToggleLedByNotification(uint32_t receivedValue)
{
	if(receivedValue == 1)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_12);
	}
	else if(receivedValue == 2)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_13);
	}
	else if(receivedValue == 3)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_14);
	}
	else if(receivedValue == 4)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_15);
	}
	else
	{
		HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14 | GPIO_PIN_15, GPIO_PIN_RESET);
	}
}

static void LedToggleSenderTask(void* pvParameters)
{
	uint32_t counter = 0;

	for(;;)
	{
		vTaskDelay(pdMS_TO_TICKS(TASK_DELAY_MS));
		counter++;
		if(counter > 6) {
			counter = 0;
		}

		if(xTaskReceiver != NULL)
		{
			xTaskNotify(xTaskReceiver, counter, eSetValueWithOverwrite);
		}
	}
}
```
