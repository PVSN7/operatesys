#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define BUFFER_SIZE 10

int buffer[BUFFER_SIZE];
int index = -1; // Indicates the current position of the buffer (-1 means empty)

pthread_mutex_t mutex;        // Mutex lock
pthread_cond_t buffer_empty;  // Condition variable for buffer empty
pthread_cond_t buffer_full;   // Condition variable for buffer full

void *producer(void *arg) {
    while (1) {
        int item = rand() % 100; // Random item to produce
        pthread_mutex_lock(&mutex);

        // Wait until the buffer is empty
        while (index != -1) {
            pthread_cond_wait(&buffer_empty, &mutex);
        }

        // Produce the item
        buffer[++index] = item;
        printf("Producer produced: %d\n", item);

        // Signal the consumer
        pthread_cond_signal(&buffer_full);
        pthread_mutex_unlock(&mutex);

        sleep(1); // Simulate work
    }
    return NULL;
}

void *consumer(void *arg) {
    while (1) {
        pthread_mutex_lock(&mutex);

        // Wait until the buffer has content
        while (index == -1) {
            pthread_cond_wait(&buffer_full, &mutex);
        }

        // Consume the item
        int item = buffer[index--];
        printf("Consumer consumed: %d\n", item);

        // Signal the producer
        pthread_cond_signal(&buffer_empty);
        pthread_mutex_unlock(&mutex);

        sleep(1); // Simulate work
    }
    return NULL;
}

int main() {
    pthread_t prod_thread, cons_thread;

    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&buffer_empty, NULL);
    pthread_cond_init(&buffer_full, NULL);

    pthread_create(&prod_thread, NULL, producer, NULL);
    pthread_create(&cons_thread, NULL, consumer, NULL);

    pthread_join(prod_thread, NULL);
    pthread_join(cons_thread, NULL);

    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&buffer_empty);
    pthread_cond_destroy(&buffer_full);

    return 0;
}
