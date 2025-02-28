/*
 * ---------------------------------------------------------------------------
 * 
 * Projeto: Escritor e leitor usando FIFO
 * 
 * Descrição:
 * 
 * 
 * 
 * 
 * Desenvolvedores:
 * João Xavier
 * Luan Cotini
 * 
 * Data de Criação: 29/11/2024
 * 
 * 
 * ---------------------------------------------------------------------------
 */

#include <zephyr.h>
#include <device.h>
#include <devicetree.h>
#include <drivers/gpio.h>
#include <sys/printk.h>
#include <sys/fifo.h>

/* define um tamanho de pilha pra ser usada pela thread e uma prioridade pra ela */
#define STACK_SIZE 1024
#define PRIORITY 5

K_FIFO_DEFINE(data_fifo); /* declara e inicializa (ta embutida dentro) uma FIFO chamada data_fifo */

struct data_item_t {
    void *fifo_reserved; /* 1st word reserved for use by fifo */
    int data;
};

struct data_item_t tx_data; /* cria uma variável tx_data do tipo data_item_t */

/* thread para escrever */
void writer_thread(void)
{
    while (1) {
        if (k_fifo_is_empty(&data_fifo)) {
            tx_data.data = 4; /* armazena o valor 4 no campo data da variável tx_data */
            printk("Escrevendo dados: %d\n", tx_data.data); /* printa na serial que tá escrevendo */
            k_fifo_put(&data_fifo, &tx_data); /* escreve na FIFO */
        }
        k_sleep(K_SECONDS(1)); /* dorme por 1 segundo */
    }
}

/* thread para ler */
void reader_thread(void)
{
    struct data_item_t *rx_data; /* cria um ponteiro para rx_data do tipo data_item_t */

    while (1) {
        rx_data = k_fifo_get(&data_fifo, K_FOREVER); /* lê o conteúdo da pilha e guarda em rx_data K_FOREVER: aguarda o mutex ficar disponível para acessar a FIFO */ 
        if (rx_data != NULL) {
            printk("Lendo dados: %d\n", rx_data->data); /* printa na serial se a FIFO não estiver vazia */
        }
        k_sleep(K_SECONDS(1)); /* dorme por 1 segundo */
    }
}

/* cria as threads */
void create_threads(){
  K_THREAD_DEFINE(writer_tid, STACK_SIZE, writer_thread, NULL, NULL, NULL, PRIORITY, 0, 0); 
  K_THREAD_DEFINE(reader_tid, STACK_SIZE, reader_thread, NULL, NULL, NULL, PRIORITY, 0, 0);
}


int main() {
    create_threads();
    while (1) {
    }
    return 0;
}

