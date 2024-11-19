#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TAM_NOME 100
#define TAM_GENERO 50
#define TAM_AUTOR 100

typedef struct {
    int ano; // Ano da obra
    char genero[TAM_GENERO]; // Gênero da obra
    char autor[TAM_AUTOR]; // Autor da obra
    char nome[TAM_NOME]; // Nome da obra
} Obra;

typedef struct Node {
    Obra obra;
    struct Node *esquerda, *direita;
} Node;

Node* raiz = NULL; // Raiz da árvore

Node* criarNo(Obra obra) {
    Node* novoNo = (Node*)malloc(sizeof(Node));
    novoNo->obra = obra;
    novoNo->esquerda = novoNo->direita = NULL;
    return novoNo;
}

void inserir(Node** raiz, Obra obra) {
    if (*raiz == NULL) {
        *raiz = criarNo(obra);
        return;
    }

    if (strcmp(obra.nome, (*raiz)->obra.nome) < 0) {
        inserir(&(*raiz)->esquerda, obra);
    } else {
        inserir(&(*raiz)->direita, obra);
    }
}

void listar(Node* raiz) 
{
    if (raiz != NULL) 
    {
        listar(raiz->esquerda);
        printf("\nAno: %d\nGênero: %s\nAutor: %s\nNome: %s\n\n", 
            raiz->obra.ano, raiz->obra.genero, raiz->obra.autor, raiz->obra.nome);
        listar(raiz->direita);
        
    }
}

void salvarObras(Node* raiz, FILE* arquivo) {
    if (raiz != NULL) {
        salvarObras(raiz->esquerda, arquivo);
        fprintf(arquivo, "%d;%s;%s;%s\n", 
            raiz->obra.ano, raiz->obra.genero, raiz->obra.autor, raiz->obra.nome);
        salvarObras(raiz->direita, arquivo);
    }
}

void carregarObras(FILE* arquivo) {
    Obra obra;
    while (fscanf(arquivo, "%d;%99[^;];%99[^;];%99[^\n]\n", 
                  &obra.ano, obra.genero, obra.autor, obra.nome) != EOF) {
        // Remove os espaços em branco e o '\n' no final dos campos, se existirem
        obra.genero[strcspn(obra.genero, "\n")] = '\0';
        obra.autor[strcspn(obra.autor, "\n")] = '\0';
        obra.nome[strcspn(obra.nome, "\n")] = '\0';

        // Insere a obra na árvore
        inserir(&raiz, obra);
    }
}



void menu() {
    printf("\n--- Menu ---\n");
    printf("1. Inserir Obra\n");
    printf("2. Listar Obras\n");
    printf("3. Modificar Obra\n");
    printf("4. Excluir Obra\n");
    printf("5. Buscar Obras\n");
    printf("6. Sair\n");
}

void salvarEmArquivo() {
    FILE *arquivo = fopen("obras.txt", "w");
    if (arquivo != NULL) {
        salvarObras(raiz, arquivo); // Salva todas as obras no arquivo
        fclose(arquivo);
        printf("\nAlterações salvas no arquivo.\n");
    } else {
        printf("\nErro ao abrir o arquivo para salvar as alterações.\n");
    }
}

void modificarObra(char* nome) {
    Node* current = raiz;

    while (current != NULL) { 
        if (strcmp(current->obra.nome, nome) == 0) { 
            printf("\n---Modificar Obra---\n");

            printf("\nDigite o novo ano (Atual é: %d): ", current->obra.ano);
            scanf("%d", &current->obra.ano);

            printf("Digite o novo gênero (Atual é: %s): ", current->obra.genero);
            getchar(); // Limpa o buffer do newline
            fgets(current->obra.genero, TAM_GENERO, stdin);
            current->obra.genero[strcspn(current->obra.genero, "\n")] = '\0'; // Remove newline

            printf("Digite o novo autor (Atual é: %s): ", current->obra.autor);
            fgets(current->obra.autor, TAM_AUTOR, stdin);
            current->obra.autor[strcspn(current->obra.autor, "\n")] = '\0'; // Remove newline

            printf("Digite o novo nome (Atual é: %s): ", current->obra.nome);
            fgets(current->obra.nome, TAM_NOME, stdin);
            current->obra.nome[strcspn(current->obra.nome, "\n")] = '\0'; // Remove newline

            printf("\nObra modificada com sucesso!\n");

            // Salvar as alterações no arquivo
            salvarEmArquivo();

            return;
        }

        if (strcmp(nome, current->obra.nome) < 0) {
            current = current->esquerda;
        } else {
            current = current->direita;
        }
    }

    printf("\nNenhuma obra encontrada com o nome '%s'.\n", nome);
}

void buscarObrasPorGenero(Node* raiz, char* genero) {
    if (raiz == NULL) {
        return; // Se o nó for nulo, retorna
    }

    // Verifica se o gênero do nó atual corresponde ao gênero procurado
    if (strcmp(raiz->obra.genero, genero) == 0) {
        printf("\nAno: %d\nGênero: %s\nAutor: %s\nNome: %s\n\n",
               raiz->obra.ano, raiz->obra.genero, raiz->obra.autor, raiz->obra.nome);
    }

    // Chama a função recursivamente para os nós da esquerda e da direita
    buscarObrasPorGenero(raiz->esquerda, genero);
    buscarObrasPorGenero(raiz->direita, genero);
}

void buscarObrasPorAno(Node* raiz, int ano) {
    if (raiz == NULL)
    {
        return; // Se o nó for nulo, retorna
    }

    // Verifica se o ano da obra corresponde ao ano procurado
    if (raiz->obra.ano == ano) {
        printf("\nAno: %d\nGênero: %s\nAutor: %s\nNome: %s\n\n",
               raiz->obra.ano, raiz->obra.genero, raiz->obra.autor, raiz->obra.nome);
    }

    // Chama a função recursivamente para os nós da esquerda e da direita
    buscarObrasPorAno(raiz->esquerda, ano);
    buscarObrasPorAno(raiz->direita, ano);
}


void buscarObrasPorAutor(Node* raiz, char* autor) {
    if (raiz == NULL) {
        return;
    }

    // Remove o '\n' da string de autor fornecida pelo usuário, se existir
    autor[strcspn(autor, "\n")] = '\0';

    // Verifica se o autor do nó atual corresponde ao autor procurado
    if (strcmp(raiz->obra.autor, autor) == 0) {
        printf("\nAno: %d\nGênero: %s\nAutor: %s\nNome: %s\n\n",
               raiz->obra.ano, raiz->obra.genero, raiz->obra.autor, raiz->obra.nome);
    }

    // Chama a função recursivamente para os nós da esquerda e da direita
    buscarObrasPorAutor(raiz->esquerda, autor);
    buscarObrasPorAutor(raiz->direita, autor);
}


void buscarObrasPorAutor_Iterativo(char* autor) {
    Node* current = raiz;
    int found = 0;

    printf("\n--- Resultados da Busca por Autor: %s ---\n", autor);

    while (current != NULL) {
        // Remove o '\n' da string de autor fornecida pelo usuário
        autor[strcspn(autor, "\n")] = '\0';

        // Verifica se o autor corresponde
        if (strcmp(current->obra.autor, autor) == 0) {
            printf("\nAno: %d\nGênero: %s\nAutor: %s\nNome: %s\n\n",
                   current->obra.ano, current->obra.genero, current->obra.autor, current->obra.nome);
            found = 1;
        }

        // Continua percorrendo a árvore
        if (strcmp(autor, current->obra.autor) < 0) {
            current = current->esquerda;
        } else {
            current = current->direita;
        }
    }

    if (!found) {
        printf("\nNenhuma obra encontrada para o autor '%s'.\n", autor);
    }
}



Node* buscarObrasPorGenero_recursivo(char* genero) {
   Node* current = raiz;
   int found = 0;

   printf("\n--- Resultados da Busca por Gênero: %s ---\n", genero);

   while (current != NULL) { 
       // Percorre a árvore para buscar todas as ocorrências
       if (strcmp(current -> obra.genero , genero ) == 0 ) { 
           printf("\nAno: %d\nGênero: %s\nAutor: %s\nNome: %s\n\n",
               current -> obra.ano,
               current -> obra.genero,
               current -> obra.autor,
               current -> obra.nome );
           found = 1; 
       }

       // Percorre a árvore em busca de outras ocorrências
       if(strcmp(genero ,current -> obra.genero ) < 0){
           current=current -> esquerda ;
       }else{
           current=current -> direita ;
       }
   }

   if (!found)
       printf("\nNenhuma obra encontrada para o gênero '%s'.\n", genero);
    return NULL; // Return NULL if not found
}

void buscarObrasPorNome(char* nome) {
   Node* current = raiz;
   int found = 0;

   printf("\n--- Resultados da Busca por Nome: %s ---\n", nome);

   while (current != NULL) { 
       // Percorre a árvore para buscar todas as ocorrências
       if (strcmp(current -> obra.nome , nome ) == 0 ) { 
           printf("\nAno: %d\nGênero: %s\nAutor: %s\nNome: %s\n\n",
               current -> obra.ano,
               current -> obra.genero,
               current -> obra.autor,
               current -> obra.nome );
           found = 1; 
       }

       // Percorre a árvore em busca de outras ocorrências
       if(strcmp(nome ,current -> obra.nome ) < 0){
           current=current -> esquerda ;
       }else{
           current=current -> direita ;
       }
   }

   if (!found)
       printf("\nNenhuma obra encontrada para o nome '%s'.\n", nome);
}

void subMenuBusca() {
   int opcao;
   do {
       printf("\n--- Sub-menu de Busca ---\n");
       printf("1. Buscar por Autor\n");
       printf("2. Buscar por Ano\n");
       printf("3. Buscar por Gênero\n");
       printf("4. Buscar por Nome da Obra\n");
       printf("5. Voltar ao Menu Principal\n");
       printf("\nEscolha uma opção: ");
       scanf("%d", &opcao);

       switch(opcao) {
           case 1: { // Buscar por Autor
               char autor[TAM_AUTOR];
               printf("\nDigite o autor: ");
               getchar(); // Limpa o buffer do newline
               fgets(autor, TAM_AUTOR, stdin);
               autor[strcspn(autor, "\n")] = '\0'; // Remove o newline

               buscarObrasPorAutor(raiz, autor);
               break;
           }


           case 2: { // Buscar por Ano
               int ano;
               printf("\nDigite o ano: ");
               scanf("%d", &ano);
               buscarObrasPorAno(raiz, ano);
               break;
           }


           case 3: { // Buscar por Gênero
               char genero[TAM_GENERO];
               printf("\nDigite o gênero: ");
               getchar(); // Limpa o buffer do newline
               fgets(genero, TAM_GENERO, stdin);
               genero[strcspn(genero, "\n")] = '\0'; // Remove newline

               printf("\n--- Resultados da Busca por Gênero: %s ---\n", genero);
               buscarObrasPorGenero(raiz, genero);
               break;
           }

           case 4: { // Buscar por Nome da Obra
               char nome[TAM_NOME];
               printf("\nDigite o nome da obra: ");
               getchar(); // Limpa o buffer do newline
               fgets(nome, TAM_NOME, stdin);
               nome[strcspn(nome, "\n")] = '\0'; // Remove newline
               buscarObrasPorNome(nome);
               break;
           }
           case 5:
               printf("\nVoltando ao Menu Principal...\n");
               break;
           default:
               printf("\nOpção inválida! Tente novamente.\n");
       }
   } while(opcao != 5); // Voltar ao menu principal
}



// Função para excluir uma obra da árvore
Node* encontrarMinimo(Node* raiz){
   while (raiz && raiz -> esquerda != NULL)
      raiz=raiz -> esquerda;

   return raiz;  
}

int excluir(Node** raiz, const char* nome) {
    if (*raiz == NULL)
        return 0;  // Não encontrou o nó

    if (strcmp(nome, (*raiz)->obra.nome) < 0)
        return excluir(&(*raiz)->esquerda, nome);

    if (strcmp(nome, (*raiz)->obra.nome) > 0)
        return excluir(&(*raiz)->direita, nome);

    // Se chegou aqui, encontrou o nó
    Node* temp;

    if ((*raiz)->esquerda == NULL) {
        temp = (*raiz)->direita;
        free(*raiz);
        *raiz = temp;
        return 1;  // Excluído com sucesso
    } else if ((*raiz)->direita == NULL) {
        temp = (*raiz)->esquerda;
        free(*raiz);
        *raiz = temp;
        return 1;  // Excluído com sucesso
    }

    temp = encontrarMinimo((*raiz)->direita);
    (*raiz)->obra = temp->obra;
    return excluir(&(*raiz)->direita, temp->obra.nome);
}





int main() {

   FILE* arquivo = fopen("obras.txt", "r");

   if (arquivo != NULL) {
       carregarObras(arquivo);
       fclose(arquivo);
   }

   int opcao;

   do {
      menu();
      printf("\nEscolha uma opção: ");
      scanf("%d", &opcao);

      switch(opcao) {
          case 1: { // Inserir Obra
              Obra novaObra;
              printf("\nDigite o ano: ");
              scanf("%d", &novaObra.ano);
              getchar(); // Limpa o buffer do newline

              printf("Digite o gênero: ");
              fgets(novaObra.genero, TAM_GENERO, stdin);
              novaObra.genero[strcspn(novaObra.genero, "\n")] = '\0'; 

              printf("Digite o autor: ");
              fgets(novaObra.autor, TAM_AUTOR, stdin);
              novaObra.autor[strcspn(novaObra.autor, "\n")] = '\0'; 

              printf("Digite o nome: ");
              fgets(novaObra.nome, TAM_NOME, stdin);
              novaObra.nome[strcspn(novaObra.nome, "\n")] = '\0'; 

              inserir(&raiz,novaObra);

              arquivo=fopen ("obras.txt","a");
              fprintf(arquivo, "%d;%s;%s;%s\n", 
                  novaObra.ano, novaObra.genero, novaObra.autor, novaObra.nome);
              fclose(arquivo );

              printf("\nObra inserida com sucesso!\n");
              break ;
          }

          case 2:{// Listar Obras
              listar(raiz);  
              break ;
          }

          case 3:{// Modificar Obra
              char nome[TAM_NOME];
              printf("\nDigite o nome da Obra que deseja modificar: ") ;
              getchar(); 
              fgets(nome,TAM_NOME ,stdin); 
              nome[strcspn(nome,"\n")]='\0'; 

              modificarObra(nome); 

          } break ;

          case 4: { // Excluir Obra
              char nome[TAM_NOME];
              printf("\nDigite o nome da Obra que deseja excluir: ");
              getchar();
              fgets(nome, TAM_NOME, stdin);
              nome[strcspn(nome, "\n")] = '\0';

              int resultado = excluir(&raiz, nome);

              if (resultado) {
                  printf("\nObra excluída com sucesso!\n");

                  // Salvar as alterações no arquivo
                  arquivo = fopen("obras.txt", "w");
                  if (arquivo != NULL) {
                      salvarObras(raiz, arquivo);
                      fclose(arquivo);
                  }
              } else {
                  printf("\nNenhuma obra encontrada com o nome '%s'.\n", nome);
              }
              break;
          }


          case 5:{// Sub-menu de Busca
             subMenuBusca(); 
          } break ;

          case 6:
             printf("\n Saindo...\n"); 
             break ;

          default:
             printf("\n Opção inválida! Tente novamente.\n"); 

      }

} while(opcao!=6); 

return 0; 
}