
#include <ctype.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define LARGO_TEXTO 100
#define LARGO_LINEA (LARGO_TEXTO * 3 + 100)   /* 3 textos + demás campos y separadores */
#define MAX_CANTIDAD 1000
#define MAX_TOTAL 1000000000.0
#define ARCHIVO_PEDIDOS "pedidos.txt"
#define ARCHIVO_ENTREGADOS "pedidos_entregados.txt"
#define ARCHIVO_PENDIENTES "pedidos_pendientes.txt"

typedef struct
{
    char cliente[LARGO_TEXTO];     /* string */
    char pedido[LARGO_TEXTO];      /* string */
    char direccion[LARGO_TEXTO];   /* string */
    char pago;                     /* char: 'E' = efectivo, 'T' = tarjeta */
    int cantidad;                  /* entero: unidades pedidas */
    double total;                  /* decimal: importe total */
    bool entregado;                /* booleano: true = ya entregado */
} Pedido;


typedef enum
{
    TODOS,
    POR_MONTO,
    POR_CLIENTE,
    POR_PAGO,
    POR_CANTIDAD
} TipoFiltro;

typedef struct
{
    TipoFiltro tipo;           /* qué condición se aplica */
    char texto[LARGO_TEXTO];   /* POR_CLIENTE: texto a buscar */
    char letra;                /* POR_PAGO: medio de pago */
    int cantidad;              /* POR_CANTIDAD: unidades */
    double monto;              /* POR_MONTO: total mínimo */
} Filtro;

int  leerTexto(const char *mensaje, char *destino, int largo);
int  leerObligatorio(const char *mensaje, char *destino, int largo);
int  leerLetra(const char *mensaje, const char *validas, char *destino);
int  convertirEntero(const char *texto, int minimo, int maximo, int *valor);
int  leerEntero(const char *mensaje, int minimo, int maximo, int *destino);
int  leerDecimal(const char *mensaje, double minimo, double maximo, double *destino);
int  pedirPedido(Pedido *p);
int  textoAPedido(const char *linea, Pedido *p);
int  leerRegistro(FILE *archivo, Pedido *p);
int  escribirRegistro(FILE *archivo, const Pedido *p);
void mostrarPedido(int numero, const Pedido *p);
void aMinuscula(char *destino, const char *origen);
int  contieneTexto(const char *texto, const char *buscado);
int  cumpleFiltro(const Pedido *p, const Filtro *filtro);
void listarSegun(const char *titulo, const Filtro *filtro);
int  mostrarMenu(void);
void agregarRegistro(void);
void listarTodo(void);
void listarPorCondicion(void);
void buscarPedidos(void);
void dividirPorEntrega(void);

int main(void)
{
    int opcion;

    do
    {
        opcion = mostrarMenu();

        switch (opcion)
        {
            case 1:
                agregarRegistro();
                break;
            case 2:
                listarTodo();
                break;
            case 3:
                listarPorCondicion();
                break;
            case 4:
                buscarPedidos();
                break;
            case 5:
                dividirPorEntrega();
                break;
            case 0:
                printf("Hasta luego.\n");
                break;
            default:
                printf("Opcion invalida.\n");
                break;
        }
    } while (opcion != 0);

    return 0;
}

 * Devuelve 1 si leyó una línea, 0 si no hay más entrada. */
int leerTexto(const char *mensaje, char *destino, int largo)
{
    char *p;
    int c;
    int n;
    int ok = 0;

    printf("%s", mensaje);
    fflush(stdout);

    if (fgets(destino, largo, stdin) != NULL)
    {
        ok = 1;

        if (strchr(destino, '\n') == NULL && !feof(stdin))
        {
            /* Línea demasiado larga: se descarta lo que sobra */
            while ((c = getchar()) != '\n' && c != EOF)
            {
            }
        }

        n = (int)strlen(destino);
        while (n > 0 && isspace((unsigned char)destino[n - 1]))
        {
            n--;
            destino[n] = '\0';
        }

        while ((p = strchr(destino, ';')) != NULL)
        {
            *p = ',';
        }
    }
    else
    {
        destino[0] = '\0';
    }

    return ok;
}

/* Pide un texto que no puede quedar vacío. Devuelve 0 si no hay más entrada. */
int leerObligatorio(const char *mensaje, char *destino, int largo)
{
    int ok = 0;

    while (!ok && leerTexto(mensaje, destino, largo))
    {
        ok = (destino[0] != '\0');

        if (!ok)
        {
            printf("Este dato es obligatorio.\n");
        }
    }

    return ok;
}

/* Pide una letra que esté en 'validas' (sin distinguir mayúsculas) y la guarda
 * en mayúscula. Devuelve 0 si no hay más entrada. */
int leerLetra(const char *mensaje, const char *validas, char *destino)
{
    char entrada[LARGO_TEXTO];
    char letra;
    int valido = 0;

    while (!valido && leerTexto(mensaje, entrada, LARGO_TEXTO))
    {
        letra = (char)toupper((unsigned char)entrada[0]);
        valido = (letra != '\0' && entrada[1] == '\0' && strchr(validas, letra) != NULL);

        if (valido)
        {
            *destino = letra;
        }
        else
        {
            printf("Dato invalido. Letras permitidas: %s\n", validas);
        }
    }

    return valido;
}

/* Convierte un texto completo en un entero entre minimo y maximo.
 * Devuelve 1 si es válido (y lo guarda en *valor), 0 si no. */
int convertirEntero(const char *texto, int minimo, int maximo, int *valor)
{
    char *fin;
    long numero = strtol(texto, &fin, 10);
    int valido = (fin != texto && *fin == '\0' && numero >= minimo && numero <= maximo);

    if (valido)
    {
        *valor = (int)numero;
    }

    return valido;
}

/* Pide un entero entre minimo y maximo y repite hasta que sea válido.
 * Devuelve 0 si no hay más entrada. */
int leerEntero(const char *mensaje, int minimo, int maximo, int *destino)
{
    char entrada[LARGO_TEXTO];
    int valido = 0;

    while (!valido && leerTexto(mensaje, entrada, LARGO_TEXTO))
    {
        valido = convertirEntero(entrada, minimo, maximo, destino);

        if (!valido)
        {
            printf("Valor invalido (entero entre %d y %d).\n", minimo, maximo);
        }
    }

    return valido;
}

/* Pide un número decimal entre minimo y maximo (acepta coma o punto).
 * Repite hasta que sea válido. Devuelve 0 si no hay más entrada. */
int leerDecimal(const char *mensaje, double minimo, double maximo, double *destino)
{
    char entrada[LARGO_TEXTO];
    char *coma;
    char *fin;
    double numero;
    int valido = 0;

    while (!valido && leerTexto(mensaje, entrada, LARGO_TEXTO))
    {
        coma = strchr(entrada, ',');

        if (coma != NULL)
        {
            *coma = '.';
        }

        numero = strtod(entrada, &fin);
        valido = (fin != entrada && *fin == '\0' && numero >= minimo && numero <= maximo);

        if (valido)
        {
            *destino = numero;
        }
        else
        {
            printf("Valor invalido (numero entre %.2f y %.2f).\n", minimo, maximo);
        }
    }

    return valido;
}

/* Pide todos los datos de un pedido. Devuelve 1 si se completó,
 * 0 si se cortó la entrada antes de terminar. */
int pedirPedido(Pedido *p)
{
    char entregado;
    int ok;

    ok = leerObligatorio("Cliente: ", p->cliente, LARGO_TEXTO)
      && leerObligatorio("Pedido: ", p->pedido, LARGO_TEXTO)
      && leerObligatorio("Direccion: ", p->direccion, LARGO_TEXTO)
      && leerLetra("Medio de pago (E=efectivo, T=tarjeta): ", "ET", &p->pago)
      && leerEntero("Cantidad de unidades: ", 1, MAX_CANTIDAD, &p->cantidad)
      && leerDecimal("Total ($): ", 0.0, MAX_TOTAL, &p->total)
      && leerLetra("Entregado? (S/N): ", "SN", &entregado);

    if (ok)
    {
        p->entregado = (entregado == 'S');
    }

    return ok;
}

/* Convierte una línea del archivo en un Pedido. Devuelve 1 si el formato es válido. */
int textoAPedido(const char *linea, Pedido *p)
{
    int entregado;
    int leidos;

    /* %99[^;] lee hasta 99 caracteres (LARGO_TEXTO - 1) antes del ';' */
    leidos = sscanf(linea, "%99[^;];%99[^;];%99[^;];%c;%d;%lf;%d",
                    p->cliente, p->pedido, p->direccion,
                    &p->pago, &p->cantidad, &p->total, &entregado);

    if (leidos == 7)
    {
        p->entregado = (entregado != 0);
    }

    return (leidos == 7);
}

/* Lee el siguiente pedido válido del archivo (ignora líneas vacías o dañadas).
 * Devuelve 0 cuando se llega al final. */
int leerRegistro(FILE *archivo, Pedido *p)
{
    char linea[LARGO_LINEA];
    int leido = 0;

    while (!leido && fgets(linea, sizeof(linea), archivo) != NULL)
    {
        leido = textoAPedido(linea, p);
    }

    return leido;
}

/* Escribe un pedido como una línea del archivo. Devuelve 1 si pudo escribirlo. */
int escribirRegistro(FILE *archivo, const Pedido *p)
{
    int escritos = fprintf(archivo, "%s;%s;%s;%c;%d;%.2f;%d\n",
                           p->cliente, p->pedido, p->direccion,
                           p->pago, p->cantidad, p->total, p->entregado ? 1 : 0);

    return (escritos > 0);
}

/* Muestra un pedido en dos líneas. */
void mostrarPedido(int numero, const Pedido *p)
{
    printf("%3d. %s - %s\n", numero, p->cliente, p->direccion);
    printf("     %s x%d | Total: $%.2f | Pago: %c | Entregado: %s\n",
           p->pedido, p->cantidad, p->total, p->pago, p->entregado ? "Si" : "No");
}

/* Copia 'origen' en 'destino' pasando todo a minúscula. */
void aMinuscula(char *destino, const char *origen)
{
    while (*origen != '\0')
    {
        *destino = (char)tolower((unsigned char)*origen);
        destino++;
        origen++;
    }

    *destino = '\0';
}

/* Devuelve 1 si 'texto' contiene a 'buscado' (sin distinguir mayúsculas).
 * Los dos textos deben tener menos de LARGO_TEXTO caracteres. */
int contieneTexto(const char *texto, const char *buscado)
{
    char textoMin[LARGO_TEXTO];
    char buscadoMin[LARGO_TEXTO];

    aMinuscula(textoMin, texto);
    aMinuscula(buscadoMin, buscado);

    return (strstr(textoMin, buscadoMin) != NULL);
}

/* Devuelve 1 si el pedido cumple la condición del filtro. */
int cumpleFiltro(const Pedido *p, const Filtro *filtro)
{
    int cumple = 1;

    switch (filtro->tipo)
    {
        case POR_MONTO:
            cumple = (p->total >= filtro->monto);
            break;
        case POR_CLIENTE:
            cumple = contieneTexto(p->cliente, filtro->texto);
            break;
        case POR_PAGO:
            cumple = (toupper((unsigned char)p->pago) == filtro->letra);
            break;
        case POR_CANTIDAD:
            cumple = (p->cantidad == filtro->cantidad);
            break;
        default:   /* TODOS: cumple siempre */
            break;
    }

    return cumple;
}

/* Recorre el archivo una sola vez y muestra los pedidos que cumplen el filtro. */
void listarSegun(const char *titulo, const Filtro *filtro)
{
    FILE *archivo = fopen(ARCHIVO_PEDIDOS, "r");
    Pedido p;
    int encontrados = 0;

    if (archivo == NULL)
    {
        printf("Todavia no existe %s. Agregue un pedido primero.\n", ARCHIVO_PEDIDOS);
    }
    else
    {
        printf("\n=== %s ===\n", titulo);

        while (leerRegistro(archivo, &p))
        {
            if (cumpleFiltro(&p, filtro))
            {
                encontrados++;
                mostrarPedido(encontrados, &p);
            }
        }

        fclose(archivo);

        if (encontrados == 0)
        {
            printf("No hay pedidos para mostrar.\n");
        }
        else
        {
            printf("Total: %d pedido(s).\n", encontrados);
        }
    }
}

/* Muestra el menú y devuelve la opción elegida (-1 si no es válida). */
int mostrarMenu(void)
{
    char entrada[LARGO_TEXTO];
    int opcion = -1;

    printf("\n===== MENU =====\n");
    printf("1. Agregar un pedido\n");
    printf("2. Listar todos los pedidos\n");
    printf("3. Listar pedidos con total mayor o igual a un monto\n");
    printf("4. Buscar pedidos por cliente, medio de pago o cantidad\n");
    printf("5. Dividir el archivo en entregados y pendientes\n");
    printf("0. Salir\n");

    if (leerTexto("Opcion: ", entrada, LARGO_TEXTO))
    {
        convertirEntero(entrada, 0, 5, &opcion);   /* si no es válida, queda en -1 */
    }
    else
    {
        opcion = 0;   /* no hay más entrada (Ctrl+D / Ctrl+Z): salir */
    }

    return opcion;
}

/* Opción 1: pide un pedido y lo agrega al final del archivo. */
void agregarRegistro(void)
{
    Pedido nuevo;
    FILE *archivo;
    int guardado = 0;
    int cerrado;

    if (pedirPedido(&nuevo))
    {
        archivo = fopen(ARCHIVO_PEDIDOS, "a");

        if (archivo != NULL)
        {
            guardado = escribirRegistro(archivo, &nuevo);
            cerrado = fclose(archivo);
            guardado = guardado && (cerrado == 0);
        }

        if (guardado)
        {
            printf("Pedido guardado en %s.\n", ARCHIVO_PEDIDOS);
        }
        else
        {
            printf("Error: no se pudo guardar el pedido.\n");
        }
    }
    else
    {
        printf("Carga cancelada: el pedido no se guardo.\n");
    }
}

/* Opción 2: lista todo el archivo. */
void listarTodo(void)
{
    Filtro filtro = { 0 };

    filtro.tipo = TODOS;
    listarSegun("TODOS LOS PEDIDOS", &filtro);
}

/* Opción 3: lista solo los pedidos cuyo total es mayor o igual a un monto. */
void listarPorCondicion(void)
{
    Filtro filtro = { 0 };

    filtro.tipo = POR_MONTO;

    if (leerDecimal("Mostrar pedidos con total mayor o igual a ($): ", 0.0, MAX_TOTAL, &filtro.monto))
    {
        listarSegun("PEDIDOS CON TOTAL MAYOR O IGUAL AL MONTO", &filtro);
    }
}

/* Opción 4: pregunta por qué campo buscar y qué dato buscar. */
void buscarPedidos(void)
{
    Filtro filtro = { 0 };
    int campo;
    int ok = 0;

    printf("\nBuscar por:\n");
    printf("1. Cliente (nombre completo o parte)\n");
    printf("2. Medio de pago\n");
    printf("3. Cantidad de unidades\n");

    if (leerEntero("Campo: ", 1, 3, &campo))
    {
        switch (campo)
        {
            case 1:
                filtro.tipo = POR_CLIENTE;
                ok = leerObligatorio("Cliente a buscar: ", filtro.texto, LARGO_TEXTO);
                break;
            case 2:
                filtro.tipo = POR_PAGO;
                ok = leerLetra("Medio de pago (E/T): ", "ET", &filtro.letra);
                break;
            default:
                filtro.tipo = POR_CANTIDAD;
                ok = leerEntero("Cantidad exacta: ", 1, MAX_CANTIDAD, &filtro.cantidad);
                break;
        }

        if (ok)
        {
            listarSegun("RESULTADOS DE LA BUSQUEDA", &filtro);
        }
    }
}

/* Opción 5: copia los pedidos del archivo original en dos archivos nuevos
 * según el campo booleano (entregados / pendientes). El original no se modifica. */
void dividirPorEntrega(void)
{
    FILE *origen = fopen(ARCHIVO_PEDIDOS, "r");
    FILE *entregados;
    FILE *pendientes;
    Pedido p;
    int cantEntregados = 0;
    int cantPendientes = 0;

    if (origen == NULL)
    {
        printf("Todavia no existe %s. Agregue un pedido primero.\n", ARCHIVO_PEDIDOS);
    }
    else
    {
        entregados = fopen(ARCHIVO_ENTREGADOS, "w");
        pendientes = fopen(ARCHIVO_PENDIENTES, "w");

        if (entregados != NULL && pendientes != NULL)
        {
            while (leerRegistro(origen, &p))
            {
                if (p.entregado)
                {
                    escribirRegistro(entregados, &p);
                    cantEntregados++;
                }
                else
                {
                    escribirRegistro(pendientes, &p);
                    cantPendientes++;
                }
            }

            printf("%s: %d pedido(s)\n", ARCHIVO_ENTREGADOS, cantEntregados);
            printf("%s: %d pedido(s)\n", ARCHIVO_PENDIENTES, cantPendientes);
        }
        else
        {
            printf("Error: no se pudieron crear los archivos de salida.\n");
        }

        if (entregados != NULL)
        {
            fclose(entregados);
        }

        if (pendientes != NULL)
        {
            fclose(pendientes);
        }

        fclose(origen);
    }
}
