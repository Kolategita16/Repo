//////////////////crc server 

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define N strlen(crc_key)

int i, j;
int count = 0;
char data[20];
char crc_key[20];
char checksum[20];

void XOR()
{
    for (j = 1; j < N; j++)
        checksum[j] = ((checksum[j] == crc_key[j]) ? '0' : '1');
}

int main()
{
    int serversocket, clientsocket, bindstatus;
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 3);
    printf("Waiting for client connection...\n");

    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
    {
        printf("connection is accepted\n");
    }

    recv(clientsocket, data, sizeof(data), 0);
    recv(clientsocket, crc_key, sizeof(crc_key), 0);
    recv(clientsocket, checksum, sizeof(checksum), 0);

    printf("Received data with checksum is: %s\n", data);

    int data_length = strlen(data);

    for (i = 0; i < N; i++)
        checksum[i] = data[i];

    do
    {
        if (checksum[0] == '1')
            XOR();
        for (j = 0; j < N - 1; j++)
            checksum[j] = checksum[j + 1];
        checksum[j] = data[i++];
    } while (i <= data_length + N - 1);

    for (i = 0; i < N; i++)
    {
        if (checksum[i] == '1')
            count++;
        else
            count = 0;
    }

    if (count < 0)
        printf("error detected\n");
    else
        printf("no error detected\n");

    char orgdata[20];
    int i;
    for (i = 0; i < data_length - 3; i++)
    {
        orgdata[i] = data[i];
    }

    printf("Orginal data is : %s\n", orgdata);
    close(serversocket);
    return 0;
}


////////////////////////////////////client

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define N strlen(crc_key)

int i, j;
char data[20];
char crc_key[20];
char checksum[20];

void XOR()
{
    for (j = 1; j < N; j++)
        checksum[j] = ((checksum[j] == crc_key[j]) ? '0' : '1');
}

int main()
{
    struct sockaddr_in serveraddress;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        printf("connection failed\n");
        return -1;
    }
    else
        printf("connection established\n");

    printf("Enter the data to be transmitted: ");
    scanf("%s", data);

    printf("Enter the CRC key: ");
    scanf("%s", crc_key);

    int data_length = strlen(data);

    for (i = data_length; i < data_length + N - 1; i++)
        data[i] = '0';
    printf("new data = %s\n", data);

    for (i = 0; i < N; i++)
        checksum[i] = data[i];

    do
    {
        if (checksum[0] == '1')
            XOR();
        for (j = 0; j < N - 1; j++)
            checksum[j] = checksum[j + 1];
        checksum[j] = data[i++];
    } while (i <= data_length + N - 1);

    printf("The check value is: %s\n", checksum);

    for (i = data_length; i < data_length + N - 1; i++)
        data[i] = checksum[i - data_length];

    printf("The final data to be sent is: %s\n", data);

    send(clientsocket, data, sizeof(data), 0);
    send(clientsocket, crc_key, sizeof(crc_key), 0);
    send(clientsocket, checksum, sizeof(checksum), 0);

    /*char error_msg[30];
    recv(clientsocket, error_msg, sizeof(error_msg), 0);
    printf("Error status: %s\n", error_msg);*/

    close(clientsocket);

    return 0;
}


//////////////////////////////////////bit stuffserver
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netdb.h>
#include <netinet/in.h>
#include <unistd.h>

void main()
{
    char data[100], b[100];
    int n, count = 0, p = -1;
    printf("\nEnter Data:");
    scanf("%s", data);
    n = strlen(data);

    char flag[10] = "01111110";
    char esc[2] = "0";
    char data1[100];
    int k = n;
    printf("\nflag =%s", flag);
    char ch[10] = "0";
    while (k % 8 != 0)
    {
        strcat(data1, ch);
        k = n + strlen(data1);
    }

    strcat(data1, data);

    n = strlen(data1);

    for (int i = 0; i < n; i++)
    {
        if (data1[i] == '0' && data1[i + 1] == '1')
        {

            k = i;
            count = -1;
            for (int j = 0; j < 9; j++)
            {

                if (data1[k] == flag[j])
                {
                    count++;

                    k++;
                }
            }
        }

        if (count == 8)
        {
            int m = 0;
            while (m < 6)
            {
                p++;
                b[p] = data1[i];
                i++;
                m++;
            }
            count = 0;
            i--;
            p++;
            m = 0;
            b[p] = '0';
            while (m < 2)
            {
                p++;
                b[p] = data1[i];
                i++;
                m++;
            }
        }

        if (count != 8)
        {

            p++;
            b[p] = data1[i];
        }
    }
    char final[200];

    strcat(final, flag);
    strcat(final, b);
    strcat(final, flag);
    printf("\n%s", final);

    int client_scoket;
    client_scoket = socket(AF_INET, SOCK_STREAM, 0);
    if (client_scoket < 0)
        printf("\nproblem in socket creation");
    struct sockaddr_in client_address;
    client_address.sin_family = AF_INET;
    client_address.sin_addr.s_addr = INADDR_ANY;
    client_address.sin_port = htons(9000);
    int con_status = connect(client_scoket, (struct sockaddr *)&client_address, sizeof(client_address));
    if (con_status < 0)
        printf("\nproblem in connection");
    write(client_scoket, final, 100);
}


/////////////////////////client bitstuff
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netdb.h>
#include <netinet/in.h>
#include <unistd.h>

void main()
{
    char data[100], b[100];
    int n, count = 0, p = -1;
    printf("\nEnter Data:");
    scanf("%s", data);
    n = strlen(data);

    char flag[10] = "01111110";
    char esc[2] = "0";
    char data1[100];
    int k = n;
    printf("\nflag =%s", flag);
    char ch[10] = "0";
    while (k % 8 != 0)
    {
        strcat(data1, ch);
        k = n + strlen(data1);
    }

    strcat(data1, data);

    n = strlen(data1);

    for (int i = 0; i < n; i++)
    {
        if (data1[i] == '0' && data1[i + 1] == '1')
        {

            k = i;
            count = -1;
            for (int j = 0; j < 9; j++)
            {

                if (data1[k] == flag[j])
                {
                    count++;

                    k++;
                }
            }
        }

        if (count == 8)
        {
            int m = 0;
            while (m < 6)
            {
                p++;
                b[p] = data1[i];
                i++;
                m++;
            }
            count = 0;
            i--;
            p++;
            m = 0;
            b[p] = '0';
            while (m < 2)
            {
                p++;
                b[p] = data1[i];
                i++;
                m++;
            }
        }

        if (count != 8)
        {

            p++;
            b[p] = data1[i];
        }
    }
    char final[200];

    strcat(final, flag);
    strcat(final, b);
    strcat(final, flag);
    printf("\n%s", final);

    int client_scoket;
    client_scoket = socket(AF_INET, SOCK_STREAM, 0);
    if (client_scoket < 0)
        printf("\nproblem in socket creation");
    struct sockaddr_in client_address;
    client_address.sin_family = AF_INET;
    client_address.sin_addr.s_addr = INADDR_ANY;
    client_address.sin_port = htons(9000);
    int con_status = connect(client_scoket, (struct sockaddr *)&client_address, sizeof(client_address));
    if (con_status < 0)
        printf("\nproblem in connection");
    write(client_scoket, final, 100);
}




///////////////////////////bytestuff server
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netdb.h>
#include <netinet/in.h>
#include <unistd.h>

void main()

{
    int server_scoket, client_socket;
    server_scoket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_scoket < 0)
    {
        printf("\nproblem in socket creation");
    }
    struct sockaddr_in server_address, client_address;
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = INADDR_ANY;
    server_address.sin_port = htons(9000);
    int bind_status = bind(server_scoket, (struct sockaddr *)&server_address, sizeof(server_address));
    if (bind_status < 0)
        printf("\nproblem in connection");
    listen(server_scoket, 5);
    int clength = sizeof(client_address);
    client_socket = accept(server_scoket, (struct sockaddr *)&client_address, &clength);
    char data[100];
    read(client_socket, data, 100);

    char b[100];
    int n, count = 0, p = -1;

    n = strlen(data);

    char flag[10] = "01111110";
    char esc[10] = "00011011";

    int k = n;

    char data1[100];
    strcpy(data1, data);

    n = strlen(data1);
    char start[10];
    start[0] = data[0];
    start[1] = data[1];
    start[2] = data[2];
    start[3] = data[3];
    start[4] = data[4];
    start[5] = data[5];
    start[6] = data[6];
    start[7] = data[7];

    int pointer = 0;

    int counter = 0;
    for (int i = 0; i < 8; i++)
    {
        if (start[i] == data[i])
        {
            counter++;
        }
    }

    if (counter == 8)
    {

        for (int i = 8; i < n; i++)

        {

            if (pointer != 1)
            {

                if (data[i] == '0' && data[i + 1] == '1')
                {

                    k = i;
                    count = -1;
                    for (int j = 0; j < 9; j++)
                    {

                        if (data[k] == flag[j])
                        {
                            count++;

                            k++;
                        }
                    }
                    if (count == 8)
                    {
                        pointer = 1;
                        break;
                    }
                }
                else
                {

                    k = i;
                    count = 0;
                    for (int j = 0; j < 9; j++)
                    {
                        if (data1[k] == esc[j])
                        {
                            count++;

                            k++;
                        }
                        else
                        {
                            break;
                        }
                    }
                }
                if (count == 8 && pointer != 1)
                {
                    i += 8;
                }

                if (pointer != 1)
                {
                    for (int j = i; j < (i + 8); j++)
                    {
                        p++;

                        b[p] = data1[j];
                    }
                    i += 7;
                }
            }
        }
    }
    else
    {
        printf("\nfalied");
    }

    char final[200];

    printf("\nb=%s", b);
}

//////////////////////////////////////byteclient


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netdb.h>
#include <netinet/in.h>
#include <unistd.h>

void main()
{
    char data[100], b[100];
    int n, count = 0, p = -1;
    printf("\nEnter Data:");
    scanf("%s", data);
    n = strlen(data);

    char flag[10] = "01111110";
    char esc[10] = "00011011";
    char data1[100];
    int k = n;
    printf("\nflag =%s", flag);
    char ch[10] = "0";
    while (k % 8 != 0)
    {
        strcat(data1, ch);
        k = n + strlen(data1);
    }

    strcat(data1, data);

    n = strlen(data1);

    for (int i = 0; i < n; i++)
    {
        if (data1[i] == '0' && data1[i + 1] == '1')
        {

            k = i;
            count = -1;
            for (int j = 0; j < 9; j++)
            {

                if (data1[k] == flag[j])
                {
                    count++;

                    k++;
                }
            }
        }
        else
        {

            k = i;
            count = -1;
            for (int j = 0; j < 9; j++)
            {
                if (data1[k] == esc[j])
                {
                    count++;
                    k++;
                }
            }
        }
        if (count == 8)
        {

            for (int j = 0; j < 8; j++)
            {
                p++;
                b[p] = esc[j];
            }
        }

        for (int j = i; j < (i + 8); j++)
        {
            p++;

            b[p] = data1[j];
        }
        i += 8;
    }
    char final[200];

    strcat(final, flag);
    strcat(final, b);
    strcat(final, flag);
    printf("\n%s", final);

    int client_scoket;
    client_scoket = socket(AF_INET, SOCK_STREAM, 0);
    if (client_scoket < 0)
        printf("\nproblem in socket creation");
    struct sockaddr_in client_address;
    client_address.sin_family = AF_INET;
    client_address.sin_addr.s_addr = INADDR_ANY;
    client_address.sin_port = htons(9000);
    int con_status = connect(client_scoket, (struct sockaddr *)&client_address, sizeof(client_address));
    if (con_status < 0)
        printf("\nproblem in connection");
    write(client_scoket, final, 100);
}





///////////////////////////// GO BACK N SERVER

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000

int main()
{
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    int serversocket;
    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int bindstatus;
    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 3);
    printf("Waiting for client connection...\n");

    int clientsocket;
    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
        printf("connection is accepted\n");

    int j = 0, f, n, ack = 0, count = 1, c = 1;
    recv(clientsocket, &n, sizeof(n), 0);
    recv(clientsocket, &f, sizeof(f), 0);
    for (int i = 0; i < n + f; i++)
    {
        if (i < f)
        {
            recv(clientsocket, &j, sizeof(j), 0);
            printf("\nbit recieved:%d", j);
        }
        else
        {
            c = 1;
            if (i == f + 2 && count != 2)
            {
                printf("\n do you want to send ack for bit=%d??y/n:", ack);
                char ans;
                scanf("%c", &ans);
                if (ans == 'n' && count != 2)
                {
                    i = ack;
                    ack -= 1;
                    count = 2;
                    c = 2;
                }
            }
            send(clientsocket, &ack, sizeof(ack), 0);
            if (c != 2 && ack < n)
                printf("\nsending ack for:%d", ack);
            if (i < n)
            {
                recv(clientsocket, &j, sizeof(j), 0);
                printf("\n bit received:%d", j);
            }

            ack++;
        }
    }

    close(serversocket);

    return 0;
}



/////////////////////GO BACK N CLIENT



#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000

int main()
{
    struct sockaddr_in serveraddress;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        printf("connection failed\n");
        return -1;
    }
    else
        printf("connection established\n");

    int n, f;
    printf("Enter the size: ");
    scanf("%d", &n);

    printf("\nEnter the frame size: ");
    scanf("%d", &f);

    send(clientsocket, &n, sizeof(n), 0);
    send(clientsocket, &f, sizeof(f), 0);

    int j = 0, ack = 0, pre = -1, count = 1, c = 1;

    for (int i = 0; i < n + f; i++)
    {
        if (i < f)
        {
            send(clientsocket, &i, sizeof(j), 0);
            printf("\nbit sent:%d", i);
        }
        else
        {

            recv(clientsocket, &ack, sizeof(j), 0);
            c = 1;
            if (ack != pre + 1 && count != 2)
            {

                i = pre + 1;
                count = 2;
                c = 2;

                printf("\n ack not received for the bit  =%d", ack + 1);
            }
            if (c != 2 && ack < n)
                printf("\n ack received:%d", ack);
            if (i < n)
            {
                send(clientsocket, &i, sizeof(ack), 0);
                printf("\nbit sent:%d", i);
            }
            pre++;
        }
    }

    close(clientsocket);

    return 0;
}






///////////////////////////HAMMING CODE SERVER

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000

int i, j;
char data[20], final_data[20], rec_data[20];

int parity_check(int *parity_index, int parity_index_size, char *rec_data)
{
    int x;
    int count = 0;
    for (i = 0; i < parity_index_size; i++)
    {
        int index = parity_index[i];
        if (rec_data[index] == '1')
            count++;
    }

    if (count % 2 != 0)
        x = 1;
    else
        x = 0;

    return x;
}

void strrev(char *rec_data)
{
    int len;
    len = strlen(rec_data);
    int start = 0, end = len - 1;
    while (start < end)
    {
        char temp = rec_data[start];
        rec_data[start] = rec_data[end];
        rec_data[end] = temp;
        start++;
        end--;
    }
}

int main()
{
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    int serversocket;
    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int bindstatus;
    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 3);
    printf("Waiting for client connection...\n");

    int clientsocket;
    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
        printf("connection is accepted\n");

    recv(clientsocket, final_data, sizeof(final_data), 0);
    printf("Received data: %s\n", final_data);

    printf("Enter the data received: ");
    scanf("%s", rec_data);

    strrev(rec_data);

    int parity_index1[] = {0, 2, 4, 6, 8, 10};
    int parity_index2[] = {1, 2, 5, 6, 9, 10};
    int parity_index3[] = {3, 4, 5, 6};
    int parity_index4[] = {7, 8, 9, 10};

    int parity_index1_size = sizeof(parity_index1) / sizeof(parity_index1[0]);
    int parity_index2_size = sizeof(parity_index2) / sizeof(parity_index2[0]);
    int parity_index3_size = sizeof(parity_index3) / sizeof(parity_index3[0]);
    int parity_index4_size = sizeof(parity_index4) / sizeof(parity_index4[0]);

    char parity_results[4];

    parity_results[3] = parity_check(parity_index1, parity_index1_size, rec_data);
    parity_results[2] = parity_check(parity_index2, parity_index2_size, rec_data);
    parity_results[1] = parity_check(parity_index3, parity_index3_size, rec_data);
    parity_results[0] = parity_check(parity_index4, parity_index4_size, rec_data);

    printf("Parity check results: P8: %d, P4: %d, P2: %d, P1: %d\n", parity_results[0], parity_results[1], parity_results[2], parity_results[3]);

    int sum = 0;
    for (i = 0; i < 4; i++)
    {
        int y = sizeof(parity_results) - (i + 1);
        sum = sum + parity_results[i] * pow(2, y);
    }

    if (sum == 0)
        printf("\nNo error\n");
    else
        printf("\nError in bit %d\n", sum);

    close(serversocket);

    return 0;
}





//////////////////////////////HAME CODE CLIENT 

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000

int i, j;
char data[20], final_data[20], rec_data[20];

void even_parity(int *parity_index, int parity_index_size, int x, char *final_data)
{
    int count = 0;
    for (i = 0; i < parity_index_size; i++)
    {
        int index = parity_index[i];
        if (final_data[index] == '1')
            count++;
    }

    if (count % 2 != 0)
        final_data[x] = '1';
    else
        final_data[x] = '0';

    printf("Final string: %s\n", final_data);
}

void strrev(char *rec_data)
{
    int len;
    len = strlen(rec_data);
    int start = 0, end = len - 1;
    while (start < end)
    {
        char temp = rec_data[start];
        rec_data[start] = rec_data[end];
        rec_data[end] = temp;
        start++;
        end--;
    }
}

int main()
{
    struct sockaddr_in serveraddress;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        printf("connection failed\n");
        return -1;
    }
    else
        printf("connection established\n");

    printf("Enter the 7 bit data: ");
    scanf("%s", data);

    int data_length = strlen(data);

    strrev(data);

    int current_data_index = 0;
    int final_data_index = 0;

    for (i = 1; i <= data_length + 4; i++)
    {
        if ((i & (i - 1)) == 0)
        {
            final_data[final_data_index] = '_';
            final_data_index++;
        }
        else
        {
            final_data[final_data_index] = data[current_data_index];
            final_data_index++;
            current_data_index++;
        }
    }

    final_data[final_data_index] = '\0';
    printf("Final data string is: %s\n", final_data);

    int parity_index1[] = {0, 2, 4, 6, 8, 10};
    int parity_index2[] = {1, 2, 5, 6, 9, 10};
    int parity_index3[] = {3, 4, 5, 6};
    int parity_index4[] = {7, 8, 9, 10};

    int parity_index1_size = sizeof(parity_index1) / sizeof(parity_index1[0]);
    int parity_index2_size = sizeof(parity_index2) / sizeof(parity_index2[0]);
    int parity_index3_size = sizeof(parity_index3) / sizeof(parity_index3[0]);
    int parity_index4_size = sizeof(parity_index4) / sizeof(parity_index4[0]);

    even_parity(parity_index1, parity_index1_size, 0, final_data);
    even_parity(parity_index2, parity_index2_size, 1, final_data);
    even_parity(parity_index3, parity_index3_size, 3, final_data);
    even_parity(parity_index4, parity_index4_size, 7, final_data);

    strrev(final_data);

    printf("The string to be transmitted is: %s\n", final_data);

    send(clientsocket, final_data, sizeof(final_data), 0);

    close(clientsocket);

    return 0;
}



////////////////////////////SELECTIVE REPEAT  SERVER

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define N 100 // Assuming N is the window size for selective repeat

int main()
{
    int serversocket, clientsocket, bindstatus;
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 100);
    printf("Waiting for client connection...\n");

    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
    {
        printf("connection is accepted\n");
    }

    char data[20];
    printf("Enter the data: ");
    scanf("%s", data);

    int count = 0, j = 1;
    int k = strlen(data);
    int window_start = 0;
    int window_end = N - 1;

    send(clientsocket, &k, sizeof(k), 0);

    while (window_start < k)
    {
        for (int i = window_start; i <= window_end && i < k; i++)
        {
            send(clientsocket, &data[i], sizeof(data[i]), 0);
        }

        for (int i = window_start; i <= window_end && i < k; i++)
        {
            recv(clientsocket, &count, sizeof(count), 0);
            if (count == j)
            {
                printf("\nAcknowledgment received for bit %d", j);
                j++;
                window_start++;
                window_end++;
            }
            else
            {
                printf("\nAcknowledgment not received for bit %d, resending", j);
                j++;
                count++;
            }
        }
    }

    close(serversocket);
    return 0;
}




//////////////SELECTIVE REPEAT CLIENT 


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define N 100 // Assuming N is the window size for selective repeat

int main()
{
    struct sockaddr_in serveraddress;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        printf("connection failed\n");
        return -1;
    }
    else
        printf("connection established\n");

    int count = 0;
    char data[20];
    int k;

    recv(clientsocket, &k, sizeof(k), 0);

    int window_start = 0;
    int window_end = N - 1;

    while (window_start < k)
    {
        for (int i = window_start; i <= window_end && i < k; i++)
        {
            recv(clientsocket, &data[i], sizeof(data[i]), 0);
            printf("Received bit %d from server\n", i + 1);
        }

        for (int i = window_start; i <= window_end && i < k; i++)
        {
            int x;
            do
            {
                printf("+ve ack - 1/-ve ack - 2: ");
                scanf("%d", &x);

                if (x == 1)
                {
                    count = i + 1; // Positive acknowledgment received
                    send(clientsocket, &count, sizeof(count), 0);
                    window_start++;
                    break; // Exit the loop after sending acknowledgment
                }
                else if (x == 2)
                {
                    // Negative acknowledgment received or invalid input
                    send(clientsocket, &count, sizeof(count), 0);
                    // Continue to the next bit without waiting for acknowledgment
                    break;
                }
                else
                {
                    printf("Invalid input. Please enter 1 or 2.\n");
                }
            } while (1); // Keep asking for acknowledgment until valid input is received
        }
    }

    printf("\nData received from the server is %s \n", data);

    close(clientsocket);
    return 0;
}


/////////////////////////sLIDING WINDOW SERVER


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000

int Tt, Tp, n, packets, Sw, get, sendack = 0;
int timer = 0;
int i, j;

int main()
{
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    int serversocket;
    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int bindstatus;
    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 3);
    printf("Waiting for client connection...\n");

    int clientsocket;
    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
        printf("connection is accepted\n");

    recv(clientsocket, &Tt, sizeof(Tt), 0);
    recv(clientsocket, &Tp, sizeof(Tp), 0);
    recv(clientsocket, &packets, sizeof(packets), 0);
    recv(clientsocket, &n, sizeof(n), 0);
    recv(clientsocket, &Sw, sizeof(Sw), 0);

    int count = 0, ack = Tt + Tp + Tt + Tp, timer2;
    while (j < Sw)
    {
        recv(clientsocket, &get, sizeof(get), 0);
        recv(clientsocket, &timer2, sizeof(timer2), 0);
        printf("\nPacket number %d received at %d ms", get, timer2);
        j++;
    }

    while (j < packets)
    {
        send(clientsocket, &ack, sizeof(ack), 0);
        ack = ack + Tt;
        sendack++;
        recv(clientsocket, &get, sizeof(get), 0);
        recv(clientsocket, &timer2, sizeof(timer2), 0);
        printf("\nPacket number %d received at %d ms\n", get, timer2);
        j++;
    }

    while (sendack < packets)
    {
        send(clientsocket, &ack, sizeof(ack), 0);
        sendack++;

        if (sendack == Sw)
            ack = ack + Tp;
        else
            ack = ack + Tt;
    }

    close(serversocket);

    return 0;
}





/////////////////////SLIDING WINDOW CLIENT



 #include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <math.h>

#define PORT 8000

int Tt, Tp, n, packets, Sw, get, sendack = 0;
int timer = 0;
int i, j;

int main()
{
    struct sockaddr_in serveraddress;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        printf("connection failed\n");
        return -1;
    }
    else
        printf("connection established\n");

    printf("Enter the transmission time in milliseconds: ");
    scanf("%d", &Tt);
    send(clientsocket, &Tt, sizeof(Tt), 0);

    printf("Enter the propagation time in milliseconds: ");
    scanf("%d", &Tp);
    send(clientsocket, &Tp, sizeof(Tp), 0);

    int packets = 1 + 2 * (Tp / Tt);
    printf("Maximum number of packets that can be sent: %d\n", packets);
    send(clientsocket, &packets, sizeof(packets), 0);

    int n = floor(log2(packets));
    printf("Number of bits in sequence number(n): %d\n", n);
    send(clientsocket, &n, sizeof(n), 0);

    int Sw = packets - n;
    printf("Number of packets in the sender's window: %d\n", Sw);
    send(clientsocket, &Sw, sizeof(Sw), 0);

    int slidingwindow[Sw];
    int a[Sw]; // Fix 3: Use the correct array size

    int time1 = 0, time2 = 0, k = 0; // Fix 4: Initialize time1 and time2
    int ack;                         // Fix 5: Declare ack variable
    int recvack = 0;
    while (j < Sw)
    {
        send(clientsocket, &j, sizeof(j), 0);
        send(clientsocket, &time2, sizeof(time2), 0);
        printf("\nPacket no. %d transferred at %d ms", j, time1);
        a[k] = j;
        k++;
        time1 = time1 + Tt;
        time2 = time2 + Tt;
        j++;
    }
    printf("\nSliding window:");
    for (int i = 0; i < Sw; i++)
        printf("%d ", a[i]);
    while (j < packets)
    {
        recv(clientsocket, &ack, sizeof(ack), 0);
        printf("\nAcknowledgement for packet no. %d received at %d ms", recvack, ack);
        for (i = 0; i < Sw; i++)
        {
            a[i] = a[i + 1];
        }
        a[i - 1] = j;
        printf("\nSliding window:");
        for (int i = 0; i < Sw; i++)
            printf("%d ", a[i]);
        recvack++;
        time1 = ack + Tt;
        time2 = time1 + Tp + Tt;
        send(clientsocket, &j, sizeof(j), 0);
        printf("\nPacket no. %d transferred at %d ms", j, time1);
        send(clientsocket, &time2, sizeof(time2), 0);
        j++;
    }
    while (recvack < packets)
    {
        recv(clientsocket, &ack, sizeof(ack), 0);
        printf("\nAcknowledgement for packet no. %d received at %d ms\n", recvack, ack);
        recvack++;
    }
    close(clientsocket);

    return 0;
}










////////////////////////////STOP AND WAIT BASIC



#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000

int i = 0, j = 0;

int main()
{
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    int serversocket;
    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        perror("socket failed");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int bindstatus;
    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        perror("binding failed");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 3);
    printf("Waiting for client connection...\n");

    int clientsocket;
    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        perror("connection is rejected by server");
        return -1;
    }
    else
        printf("connection is accepted\n");

    int Tt, Tp, datasize, timer = 0;
    recv(clientsocket, &Tt, sizeof(Tt), 0);
    recv(clientsocket, &Tp, sizeof(Tp), 0);
    recv(clientsocket, &datasize, sizeof(datasize), 0);

    int data[datasize];

    while (i < datasize)
    {
        recv(clientsocket, &data[i], sizeof(data[i]), 0);
        recv(clientsocket, &timer, sizeof(timer), 0);
        printf("Data packet with data %d received at %d\n", data[i], timer);

        timer = timer + Tt + Tp;
        send(clientsocket, &timer, sizeof(timer), 0);

        i++;
    }

    close(serversocket);

    return 0;
}




///////////////////STOP AND WAIT BASIC CLIENT



#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000

int i = 0, j = 0;

int main()
{
    struct sockaddr_in serveraddress;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        perror("socket creation failed");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        perror("connection failed");
        return -1;
    }
    else
        printf("connection established\n");

    int Tt, Tp, datasize;
    printf("Enter the Transmission time and propagation time: ");
    scanf("%d %d", &Tt, &Tp);
    send(clientsocket, &Tt, sizeof(Tt), 0);
    send(clientsocket, &Tp, sizeof(Tp), 0);

    printf("Enter the size of data to be transmitted: ");
    scanf("%d", &datasize);
    send(clientsocket, &datasize, sizeof(datasize), 0);

    int data[datasize];
    printf("Enter the data bits: ");
    for (i = 0; i < datasize; i++)
    {
        scanf("%d", &data[i]);
    }

    int timer = 0;
    for (i = 0; i < datasize; i++)
    {
        timer = timer + Tt + Tp;
        printf("%d", timer);
        send(clientsocket, &data[i], sizeof(data[i]), 0);
        send(clientsocket, &timer, sizeof(timer), 0);

        recv(clientsocket, &timer, sizeof(timer), 0);
        printf("Ack received at %d ms\n", timer);
    }

    close(clientsocket);

    return 0;
}





//////////////////////////////////STOP AND WAIT ARQ SERVER


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define N

int main()
{
    int serversocket, clientsocket, bindstatus;
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 100);
    printf("Waiting for client connection...\n");

    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
    {
        printf("connection is accepted\n");
    }

    char data[20];
    printf("Enter the data: ");
    scanf("%s", data);

    int count = 0, j = 1;
    int k = strlen(data);

    send(clientsocket, &k, sizeof(k), 0);

    for (int i = 0; i < k; i++)
    {
        send(clientsocket, &data[i], sizeof(data[i]), 0);
        recv(clientsocket, &count, sizeof(count), 0);
        // j++; working partially, keep if nothing works otherwise j=1 and 2 j++ after printf in if
        if (count == j)
        {
            printf("\nAcknowledgement received for bit %d", j);
            j++;
        }
        else if (count != j)
        {
            printf("\nAcknowledgement not received for bit %d resending", j);
            j++;
        }
    }
    close(serversocket);
    return 0;
}



////////////////////////STOP AND WAIT ARQ  CLIENT 



#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define N

int main()
{
    struct sockaddr_in serveraddress;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        printf("connection failed\n");
        return -1;
    }
    else
        printf("connection established\n");

    int count = 0;
    char data[20];
    int k;

    recv(clientsocket, &k, sizeof(k), 0);

    for (int i = 0; i < k; i++)
    {
        recv(clientsocket, &data[i], sizeof(data[i]), 0);
        printf("Received bit %d from server\n", i + 1);

        int x;
        do
        {
            printf("+ve ack - 1/-ve ack - 2: ");
            scanf("%d", &x);

            if (x == 1)
            {
                count++;
                send(clientsocket, &count, sizeof(count), 0);
                break; // Exit the loop after sending acknowledgment
            }
            else if (x == 2)
            {

                send(clientsocket, &count, sizeof(count), 0);
                count++;
                // Continue to the next bit without waiting for acknowledgment
                break;
            }
            else
            {
                printf("Invalid input. Please enter 1 or 2.\n");
            }
        } while (1); // Keep asking for acknowledgment until valid input is received
    }

    printf("\nData received from the server is %s \n", data);

    close(clientsocket);
    return 0;
}


////////////////////////////////// CALCULATOR SERVER


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8080

int calculate(int a, int b, char operator)
{
    switch (operator)
    {
    case '+':
        return a + b;
    case '-':
        return a - b;
    case '*':
        return a * b;
    case '/':
        return a / b;
    default:
        return -1;
    }
}

int main()
{
    int serversocket, clientsocket, bindstatus;
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);

    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 3);
    printf("Waiting for client connection...\n");

    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
    {
        printf("connection is accepted\n");
    }

    int num1, num2, result;
    char operator;

    recv(clientsocket, &num1, sizeof(num1), 0);
    recv(clientsocket, &num2, sizeof(num2), 0);
    recv(clientsocket, &operator, sizeof(operator), 0);

    result = calculate(num1, num2, operator);

    send(clientsocket, &result, sizeof(result), 0);

    close(serversocket);
    return 0;
}




//////////////////////////CALCULATOR CLIENT


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8080

int main()
{
    int clientsocket;
    struct sockaddr_in serveraddress;
    int connection_status;

    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status == -1)
    {
        printf("connection failed\n");
        return -1;
    }

    int num1, num2, result;
    char operator;

    printf("Enter first number: ");
    scanf("%d", &num1);
    printf("Enter second number: ");
    scanf("%d", &num2);
    printf("Enter operator (+, -, *, /): ");
    scanf(" %c", &operator);

    send(clientsocket, &num1, sizeof(num1), 0);
    send(clientsocket, &num2, sizeof(num2), 0);
    send(clientsocket, &operator, sizeof(operator), 0);

    recv(clientsocket, &result, sizeof(result), 0);

    printf("Result received from server: %d\n", result);

    close(clientsocket);
    return 0;
}


///////////////////COM SERVER

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define MAX_MESSAGE_SIZE 1024

int main()
{
    int serversocket, clientsocket, bindstatus;
    char servermessage[256];
    struct sockaddr_in serveraddress, clientaddress;
    int client_address_len = sizeof(clientaddress);
    char message[MAX_MESSAGE_SIZE];

    serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    bindstatus = bind(serversocket, (const struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (bindstatus < 0)
    {
        printf("binding failed\n");
        return -1;
    }
    else
    {
        printf("binding is successful\n");
    }

    listen(serversocket, 3);
    printf("Waiting for client connection...\n");

    clientsocket = accept(serversocket, (struct sockaddr *)&clientaddress, (socklen_t *)&client_address_len);
    if (clientsocket < 0)
    {
        printf("connection is rejected by server\n");
        return -1;
    }
    else
    {
        printf("connection is accepted\n");
    }

    while (1)
    {
        // Receive a message from the client
        memset(message, 0, MAX_MESSAGE_SIZE);

        printf("Client: %s", message);
        if (strcmp(message, "bye\n") == 0)
        {
            printf("Client said goodbye. Exiting...\n");
            close(clientsocket);
            close(serversocket);
            break;
        }

        // Send a response to the client
        printf("Server: ");
        fgets(message, sizeof(message), stdin); // Read a message from the server
        if (strcmp(message, "bye\n") == 0)
        {
            printf("Client said goodbye. Exiting...\n");
            close(clientsocket);
            close(serversocket);
            break;
        }
        send(clientsocket, message, strlen(message), 0);
    }
    close(serversocket);

    return 0;
}

///////////COM CLIENT 



#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8000
#define MAX_MESSAGE_SIZE 1024

int main()
{
    struct sockaddr_in serveraddress;
    char message[MAX_MESSAGE_SIZE];

    int serversocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serversocket < 0)
    {
        printf("socket failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int clientsocket;
    clientsocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientsocket < 0)
    {
        printf("socket creation failed\n");
        return -1;
    }

    serveraddress.sin_family = AF_INET;
    serveraddress.sin_port = htons(PORT);
    serveraddress.sin_addr.s_addr = INADDR_ANY;

    int connection_status;
    connection_status = connect(clientsocket, (struct sockaddr *)&serveraddress, sizeof(serveraddress));
    if (connection_status < 0)
    {
        printf("connection failed\n");
        return -1;
    }
    else
        printf("connection established\n");

    while (1)
    {
        // Send a message to the server
        printf("Client: ");
        fgets(message, sizeof(message), stdin); // Read a message from the client
        if (strcmp(message, "bye\n") == 0)
        {
            printf("Server said goodbye. Exiting...\n");
            close(clientsocket);
            close(serversocket);
            break;
        }
        send(clientsocket, message, strlen(message), 0);

        // Receive a response from the server
        memset(message, 0, MAX_MESSAGE_SIZE); // used to set a block of memory to data(name,valueineachcell,size)

        printf("Server: %s", message);

        if (strcmp(message, "bye\n") == 0)
        {
            printf("Server said goodbye. Exiting...\n");
            close(clientsocket);
            close(serversocket);
            break;
        }
    }

    close(clientsocket);

    return 0;
}









