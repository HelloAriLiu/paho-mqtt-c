# paho-mqtt-c
paho c的MQTT客户端连接流程整理记录

## 说明
>接口整理：
>>连接：__`int mqtt_connect(void)`__  
>>订阅：__`int mqtt_subscribe(void)`__  
>>发布：__`int mqtt_publish(char *pubMsg, int pubMsgLen)`__  
>>状态机：__`void mqtt_state_proc(void)`__  


### 示例
```
/*
 * @Description :  
 * @FilePath: /paho-test/examples/my-mqtt-test.c
 * @Author:  LR
 * @Date: 2021-07-08 17:04:33
 * 
 * https://www.eclipse.org/paho/files/mqttdoc/MQTTClient/html/index.html
 * 
 * https://www.eclipse.org/paho/index.php?page=clients/c/index.php
 * 
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include "../src/MQTTClient.h"

#define MQTT_STATE_DISCONN 0
#define MQTT_STATE_SUB 1
#define MQTT_STATE_PUB 2
char MQTT_STATE = 0; //Mqtt状态机

#define QOS 1
#define TIMEOUT 10000L

MQTTClient client;
MQTTClient_deliveryToken token;
MQTTClient_connectOptions conn_opts = MQTTClient_connectOptions_initializer;

/**
 * @brief :  表示 MQTT 消息的值。发布消息时回调，传递令牌将返回给客户端应用程序。然后可以使用令牌来检查消息是否已成功传送到其目的地
 * @param {void} *context
 * @param {MQTTClient_deliveryToken} dt
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 15:05:02
 */
volatile MQTTClient_deliveryToken deliveredtoken;
void mqtt_delivered_callback(void *context, MQTTClient_deliveryToken dt)
{
    printf("Message with token value %d delivery confirmed\n", dt);
   // deliveredtoken = dt;
}

/**
 * @brief :  这是一个回调函数。客户端应用程序必须提供此函数的实现以启用消息的异步接收。
 * 该函数通过将其作为参数传递给MQTTClient_setCallbacks()来向客户端库注册。当从服务器收到与客户端订阅匹配的新消息时，客户端库会调用它。
 * 此函数在与运行客户端应用程序的线程不同的线程上执行。 
 * @param {void} *context
 * @param {char} *topicName
 * @param {int} topicLen
 * @param {MQTTClient_message} *message
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 15:04:23
 */
int mqtt_msgarrvd_callback(void *context, char *topicName, int topicLen, MQTTClient_message *message)
{
    char *payloadptr;
    int payloadlen;
    payloadptr = message->payload;
    payloadlen = message->payloadlen;
    printf("topic[%s] sublishMessage [%d]:%s  \n\n", topicName, payloadlen, payloadptr);
    //Todo:填入接收buff

    MQTTClient_freeMessage(&message);
    MQTTClient_free(topicName);
    return 1;
}

/**
 * @brief :  断开连接的回调处理
 * @param {void} *context
 * @param {char} *cause
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 15:05:37
 */
void mqtt_connlost_callback(void *context, char *cause)
{
    printf("\nConnection lost\n");
    printf("     cause: %s\n", cause);
    MQTTClient_disconnect(client, 10000);
    MQTTClient_destroy(&client);
    MQTT_STATE = MQTT_STATE_DISCONN;
}

/**
 * @brief :  mqtt-连接处理
 * @param {*}
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 16:00:38
 */
int mqtt_connect(void)
{
    int rc;
    // conn_opts.username = "";
    // conn_opts.password = "";
    conn_opts.keepAliveInterval = 20;
    conn_opts.cleansession = 1;

    const char *ADDRESS = "mq.tongxinmao.com:18830";
    const char *CLIENTID = "MQTTXTTD-2020IJWITO1";

    if ((rc = MQTTClient_create(&client, ADDRESS, CLIENTID, MQTTCLIENT_PERSISTENCE_NONE, NULL)) != MQTTCLIENT_SUCCESS)
    {
        printf("Failed to connect, return code %d\n", rc);

        return MQTTCLIENT_FAILURE;
    }

    if ((rc = MQTTClient_setCallbacks(client, NULL, mqtt_connlost_callback, mqtt_msgarrvd_callback, mqtt_delivered_callback)) != MQTTCLIENT_SUCCESS)
    {
        printf("Failed to connect, return code %d\n", rc);
        MQTTClient_destroy(&client);
        return MQTTCLIENT_FAILURE;
    }

    if ((rc = MQTTClient_connect(client, &conn_opts)) != MQTTCLIENT_SUCCESS)
    {
        printf("Failed to connect, return code %d\n", rc);
        MQTTClient_destroy(&client);
        return MQTTCLIENT_FAILURE;
    }

    printf("MQTTClient_connect  OK    \n");
    return MQTTCLIENT_SUCCESS;
}

/**
 * @brief :  mqtt-订阅消息
 * @param {*}
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 16:14:44
 */
int mqtt_subscribe(void)
{
    int rc;
    const char *sub_topic = "/public/TEST/2";

    if ((rc = MQTTClient_subscribe(client, sub_topic, QOS)) != MQTTCLIENT_SUCCESS)
    {
        printf("Failed to subscribe, return code %d\n", rc);
        return MQTTCLIENT_FAILURE;
    }

    return MQTTCLIENT_SUCCESS;
}

/**
 * @brief :  mqtt 发布消息
 * @param {char} *pubMsg
 * @param {int} pubMsgLen
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 16:00:29
 */
int mqtt_publish(char *pubMsg, int pubMsgLen)
{
    MQTTClient_message pubmsg = MQTTClient_message_initializer;
    pubmsg.payload = pubMsg;
    pubmsg.payloadlen = pubMsgLen;
    pubmsg.qos = QOS;
    pubmsg.retained = 0;
    deliveredtoken = 0;
    const char *pub_topic = "/public/TEST/1";

    if (MQTTClient_publishMessage(client, pub_topic, &pubmsg, &token) != MQTTCLIENT_SUCCESS)
    {
        printf("MQTTClient_publishMessage of error   \n");
        return MQTTCLIENT_FAILURE;
    }

    printf("MQTTClient_publishMessage [%d]:%s    \n", pubMsgLen, pubMsg);
    return MQTTCLIENT_SUCCESS;
}

/**
 * @brief :  mqtt处理状态机
 * @param {*}
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 16:51:36
 */
void mqtt_state_proc(void)
{
    switch (MQTT_STATE)
    {
    default:
    case MQTT_STATE_DISCONN:
    {
        if (mqtt_connect() == MQTTCLIENT_SUCCESS)
        {
            MQTT_STATE = MQTT_STATE_SUB;
        }
    }
    break;
    case MQTT_STATE_SUB:
    {
        if (mqtt_subscribe() == MQTTCLIENT_SUCCESS)
        {
            MQTT_STATE = MQTT_STATE_PUB;
        }
        else
        {
            MQTT_STATE = MQTT_STATE_DISCONN;
        }
    }
    break;
    case MQTT_STATE_PUB:
    {
        //Todo:检测发送数据缓冲区，发送数据pub
        mqtt_publish("test---123   ", 14);
    }
    break;
    }
}

/**
 * @brief :  测试用
 * @param {*}
 * @return {*}
 * @author: LR
 * @Date: 2021-07-19 17:20:01
 */
int main(void)
{
    while (1)
    {
        mqtt_state_proc();
        sleep(2);
    }
} 
```
