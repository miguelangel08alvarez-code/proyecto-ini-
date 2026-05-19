
////////////////////////////////////////////
// SIMULADOR DE BATALLAS - GRAN CARPA DE CENIZA
// PROYECTO FINAL - INTRODUCCION A LA INFORMATICA
// PROFESOR: PABLO LUCENA - IUJO
////////////////////////////////////////////

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>

////////////////////////////////////////////
// ESTRUCTURA: Personaje
// PROPOSITO: Almacena todos los atributos de un personaje
////////////////////////////////////////////
struct Personaje {
    int id;              // Identificador unico
    char nombre[20];     // Nombre del personaje
    char reino[10];      // Ceniza, Sombra, Vacio, Luz
    int hp;              // Vida actual
    int hp_max;          // Vida maxima
    int def;             // Defensa actual (escudo)
    int def_max;         // Defensa maxima
    int atq;             // Ataque base
    int ce;              // Costo de energia (para escalar daño)
};

////////////////////////////////////////////
// VARIABLES GLOBALES
////////////////////////////////////////////
struct Personaje catalogo[20];  // Arreglo de hasta 20 personajes
int totalPersonajes = 12;       // Empieza con 12 (los base)
struct Personaje equipo1[5];    // Equipo del Jugador 1
struct Personaje equipo2[5];    // Equipo del Jugador 2
int danioTotal1 = 0;            // Daño acumulado por Jugador 1
int danioTotal2 = 0;            // Daño acumulado por Jugador 2

////////////////////////////////////////////
// PROTOTIPOS DE FUNCIONES
////////////////////////////////////////////
void cargarCatalogoBase();
void mostrarCatalogo();
void crearPersonaje();
void formarEquipo(struct Personaje equipo[5], int num);
void menuCarta(struct Personaje equipo[5], int num);
float calcularDaño(struct Personaje at, struct Personaje df);
float ventaja(char a[10], char d[10]);
void batalla();
void estadisticas();
int menu();

////////////////////////////////////////////
// FUNCION: main
// PROPOSITO: Programa principal, muestra el menu y llama a las funciones
// PARAMETROS: Ninguno
// RETORNO: 0 (ejecucion exitosa)
////////////////////////////////////////////
int main()
{
    int opcion;
    srand(time(NULL));      // Inicializa numeros aleatorios
    cargarCatalogoBase();   // Carga los 12 personajes iniciales
    
    do {
        opcion = menu();
        switch(opcion) {
            case 1: crearPersonaje(); break;
            case 2: mostrarCatalogo(); break;
            case 3: 
                printf("\nJUGADOR 1\n"); formarEquipo(equipo1, 1);
                printf("\nJUGADOR 2\n"); formarEquipo(equipo2, 2);
                break;
            case 4:
                printf("\nJUGADOR 1\n"); menuCarta(equipo1, 1);
                printf("\nJUGADOR 2\n"); menuCarta(equipo2, 2);
                break;
            case 5: batalla(); break;
            case 6: estadisticas(); break;
            case 7: printf("Saliendo...\n"); break;
            default: printf("Opcion no valida\n");
        }
    } while(opcion != 7);
    
    return 0;   // Fin del programa
}

////////////////////////////////////////////
// FUNCION: menu
// PROPOSITO: Muestra las opciones y lee la eleccion del usuario
// PARAMETROS: Ninguno
// RETORNO: Entero con la opcion elegida (1-7)
////////////////////////////////////////////
int menu()
{
    int op;
    printf("\n=== MENU ===\n");
    printf("1.Crear personaje\n");
    printf("2.Ver catalogo\n");
    printf("3.Formar equipos\n");
    printf("4.Usar carta potenciacion\n");
    printf("5.Iniciar batalla\n");
    printf("6.Ver estadisticas\n");
    printf("7.Salir\n");
    printf("Opcion: ");
    scanf("%d", &op);
    return op;
}

////////////////////////////////////////////
// FUNCION: cargarCatalogoBase
// PROPOSITO: Carga los 12 personajes fijos del juego
// PARAMETROS: Ninguno
// RETORNO: Nada
////////////////////////////////////////////
void cargarCatalogoBase()
{
    struct Personaje p1 = {1,"Hisoka","Ceniza",40,40,25,25,35,5};
    struct Personaje p2 = {2,"Mikhail","Ceniza",50,50,25,25,25,4};
    struct Personaje p3 = {3,"Castiel","Ceniza",35,35,20,20,45,5};
    struct Personaje p4 = {4,"Lysander","Sombra",45,45,25,25,30,4};
    struct Personaje p5 = {5,"Espectador","Sombra",50,50,30,30,20,2};
    struct Personaje p6 = {6,"SombraFugitiva","Sombra",35,35,25,25,40,3};
    struct Personaje p7 = {7,"Ciel","Vacio",40,40,25,25,35,4};
    struct Personaje p8 = {8,"ActorEnFuga","Vacio",50,50,25,25,25,3};
    struct Personaje p9 = {9,"CuerpoSinSombra","Vacio",35,35,25,25,40,4};
    struct Personaje p10 = {10,"AgenteDelVacio","Luz",45,45,25,25,30,3};
    struct Personaje p11 = {11,"PortadorDelReloj","Luz",50,50,25,25,25,4};
    struct Personaje p12 = {12,"Instructor","Luz",55,55,25,25,20,2};
    
    catalogo[0]=p1; catalogo[1]=p2; catalogo[2]=p3;
    catalogo[3]=p4; catalogo[4]=p5; catalogo[5]=p6;
    catalogo[6]=p7; catalogo[7]=p8; catalogo[8]=p9;
    catalogo[9]=p10; catalogo[10]=p11; catalogo[11]=p12;
}

////////////////////////////////////////////
// FUNCION: mostrarCatalogo
// PROPOSITO: Muestra todos los personajes disponibles
// PARAMETROS: Ninguno
// RETORNO: Nada
////////////////////////////////////////////
void mostrarCatalogo()
{
    printf("\n--- CATALOGO ---\n");
    for(int i=0;i<totalPersonajes;i++) {
        printf("%d.%s[%s] V:%d D:%d A:%d E:%d\n",
               catalogo[i].id, catalogo[i].nombre, catalogo[i].reino,
               catalogo[i].hp, catalogo[i].def, catalogo[i].atq, catalogo[i].ce);
    }
}

////////////////////////////////////////////
// FUNCION: crearPersonaje
// PROPOSITO: Crea un nuevo personaje con formulas matematicas
// PARAMETROS: Ninguno
// RETORNO: Nada
////////////////////////////////////////////
void crearPersonaje()
{
    if(totalPersonajes >= 20) {
        printf("Catalogo lleno\n");
        return;
    }
    
    struct Personaje nuevo;
    int opReino, nivel;
    
    printf("Nombre: ");
    scanf("%s", nuevo.nombre);
    
    printf("Reino (1.Ceniza 2.Sombra 3.Vacio 4.Luz): ");
    scanf("%d", &opReino);
    if(opReino==1) strcpy(nuevo.reino,"Ceniza");
    else if(opReino==2) strcpy(nuevo.reino,"Sombra");
    else if(opReino==3) strcpy(nuevo.reino,"Vacio");
    else strcpy(nuevo.reino,"Luz");
    
    printf("Nivel (1 a 10): ");
    scanf("%d", &nivel);
    if(nivel<1) nivel=1;
    if(nivel>10) nivel=10;
    
    // FORMULAS MATEMATICAS (polinomios y potenciacion)
    nuevo.hp = 40 + (nivel * 4);
    nuevo.hp_max = nuevo.hp;
    nuevo.atq = 20 + (nivel * 3) + (nivel * nivel * 0.3);
    nuevo.def = 15 + (nivel * 2) + (nivel * 0.5);
    nuevo.def_max = nuevo.def;
    nuevo.ce = 3 + (nivel / 2);
    nuevo.id = totalPersonajes + 1;
    
    printf("Personaje creado! HP:%d ATQ:%d DEF:%d\n", nuevo.hp, nuevo.atq, nuevo.def);
    catalogo[totalPersonajes] = nuevo;
    totalPersonajes++;
}

////////////////////////////////////////////
// FUNCION: formarEquipo
// PROPOSITO: Permite al jugador elegir 5 personajes del catalogo
// PARAMETROS: equipo (arreglo a llenar), num (numero de jugador)
// RETORNO: Nada
////////////////////////////////////////////
void formarEquipo(struct Personaje equipo[5], int num)
{
    int ids[5], id, cont=0;
    mostrarCatalogo();
    while(cont<5) {
        printf("ID %d: ", cont+1);
        scanf("%d",&id);
        int encontrado=0, repetido=0;
        for(int i=0;i<totalPersonajes;i++) {
            if(catalogo[i].id==id) {
                for(int j=0;j<cont;j++) if(ids[j]==id) repetido=1;
                if(repetido==0) {
                    equipo[cont]=catalogo[i];
                    ids[cont]=id;
                    cont++;
                    encontrado=1;
                } else printf("Repetido\n");
                break;
            }
        }
        if(!encontrado) printf("ID invalido\n");
    }
    printf("Equipo %d listo\n", num);
}

////////////////////////////////////////////
// FUNCION: menuCarta
// PROPOSITO: Aplica carta potenciadora a personajes de un reino
// PARAMETROS: equipo (arreglo de personajes), num (jugador)
// RETORNO: Nada
////////////////////////////////////////////
void menuCarta(struct Personaje equipo[5], int num)
{
    int op;
    char reinoC[10];
    float a=1, h=1, d=1;
    printf("Cartas:\n");
    printf("1.LlamaEterna(Ceniza) +25 ATQ -15 DEF\n");
    printf("2.MantoOscuro(Sombra) +20 HP -10 ATQ\n");
    printf("3.AbismoInterior(Vacio) +30 DEF\n");
    printf("4.LuzDeVerdad(Luz) +25 ATQ -15 HP\n");
    printf("0.Ninguna\n");
    printf("Opcion: ");
    scanf("%d",&op);
    
    if(op==1){strcpy(reinoC,"Ceniza"); a=1.25; d=0.85;}
    else if(op==2){strcpy(reinoC,"Sombra"); h=1.20; a=0.90;}
    else if(op==3){strcpy(reinoC,"Vacio"); d=1.30;}
    else if(op==4){strcpy(reinoC,"Luz"); a=1.25; h=0.85;}
    else {printf("Sin carta\n"); return;}
    
    for(int i=0;i<5;i++) {
        if(strcmp(equipo[i].reino,reinoC)==0) {
            equipo[i].atq = equipo[i].atq * a;
            equipo[i].hp = equipo[i].hp * h;
            equipo[i].hp_max = equipo[i].hp_max * h;
            equipo[i].def = equipo[i].def * d;
            equipo[i].def_max = equipo[i].def_max * d;
            printf("%s potenciado\n", equipo[i].nombre);
        }
    }
}

////////////////////////////////////////////
// FUNCION: ventaja
// PROPOSITO: Calcula la ventaja elemental entre dos reinos
// PARAMETROS: a (reino atacante), d (reino defensor)
// RETORNO: 1.5 (fuerte), 0.8 (debil), 1.0 (normal)
////////////////////////////////////////////
float ventaja(char a[10], char d[10])
{
    if(strcmp(a,"Ceniza")==0 && strcmp(d,"Sombra")==0) return 1.5;
    if(strcmp(a,"Sombra")==0 && strcmp(d,"Vacio")==0) return 1.5;
    if(strcmp(a,"Vacio")==0 && strcmp(d,"Luz")==0) return 1.5;
    if(strcmp(a,"Luz")==0 && strcmp(d,"Ceniza")==0) return 1.5;
    if(strcmp(a,"Ceniza")==0 && strcmp(d,"Luz")==0) return 0.8;
    if(strcmp(a,"Sombra")==0 && strcmp(d,"Ceniza")==0) return 0.8;
    if(strcmp(a,"Vacio")==0 && strcmp(d,"Sombra")==0) return 0.8;
    if(strcmp(a,"Luz")==0 && strcmp(d,"Vacio")==0) return 0.8;
    return 1.0;
}

////////////////////////////////////////////
// FUNCION: calcularDaño
// PROPOSITO: Calcula el daño usando polinomio y ventaja elemental
// PARAMETROS: at (atacante), df (defensor)
// RETORNO: Daño final (float)
////////////////////////////////////////////
float calcularDaño(struct Personaje at, struct Personaje df)
{
    float base = (2 * (at.atq/10.0)*(at.atq/10.0)) + (3*(at.atq/10.0)) + 5;
    base = base / (at.ce + 1);
    base = base * 3;
    float daño = base * ventaja(at.reino, df.reino);
    if(daño < 15) daño = 15;
    if(daño > 60) daño = 60;
    return daño;
}

////////////////////////////////////////////
// FUNCION: batalla
// PROPOSITO: Simula la batalla por turnos entre dos equipos
// PARAMETROS: Ninguno (usa globales equipo1, equipo2)
// RETORNO: Nada
////////////////////////////////////////////
void batalla()
{
    int i1=0, i2=0, turno=1;
    float d;
    int resto;
    
    for(int i=0;i<5;i++) {
        equipo1[i].hp = equipo1[i].hp_max;
        equipo1[i].def = equipo1[i].def_max;
        equipo2[i].hp = equipo2[i].hp_max;
        equipo2[i].def = equipo2[i].def_max;
    }
    
    printf("\n--- BATALLA ---\n");
    
    while(i1<5 && i2<5) {
        printf("\n%s[V:%d D:%d] vs %s[V:%d D:%d]\n", 
               equipo1[i1].nombre, equipo1[i1].hp, equipo1[i1].def,
               equipo2[i2].nombre, equipo2[i2].hp, equipo2[i2].def);
        
        if(turno==1) {
            d = calcularDaño(equipo1[i1], equipo2[i2]);
            danioTotal1 += d;
            printf("J1 ataca: %.0f\n", d);
            if(d <= equipo2[i2].def) {
                equipo2[i2].def -= d;
            } else {
                resto = d - equipo2[i2].def;
                equipo2[i2].def = 0;
                equipo2[i2].hp -= resto;
            }
            turno=2;
        } else {
            d = calcularDaño(equipo2[i2], equipo1[i1]);
            danioTotal2 += d;
            printf("J2 ataca: %.0f\n", d);
            if(d <= equipo1[i1].def) {
                equipo1[i1].def -= d;
            } else {
                resto = d - equipo1[i1].def;
                equipo1[i1].def = 0;
                equipo1[i1].hp -= resto;
            }
            turno=1;
        }
        
        if(equipo2[i2].hp <= 0) {
            printf("%s muere\n", equipo2[i2].nombre);
            i2++;
        }
        if(equipo1[i1].hp <= 0) {
            printf("%s muere\n", equipo1[i1].nombre);
            i1++;
        }
    }
    
    printf("\n--- RESULTADO BATALLA ---\n");
    if(i1>=5 && i2>=5) printf("EMPATE\n");
    else if(i1>=5) printf("JUGADOR 2 GANA\n");
    else printf("JUGADOR 1 GANA\n");
}

////////////////////////////////////////////
// FUNCION: estadisticas
// PROPOSITO: Muestra el daño total causado por cada jugador
// PARAMETROS: Ninguno
// RETORNO: Nada
////////////////////////////////////////////
void estadisticas()
{
    printf("\n--- ESTADISTICAS DE DAÑO ---\n");
    printf("Daño Jugador 1: %d\n", danioTotal1);
    printf("Daño Jugador 2: %d\n", danioTotal2);
    if(danioTotal1 > danioTotal2) {
        printf("Mayor daño: Jugador 1\n");
    } else if(danioTotal2 > danioTotal1) {
        printf("Mayor daño: Jugador 2\n");
    } else {
        printf("Empate en daño\n");
    }
}