### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:


    * Para calcular el 1000000 número de la secuencia de Fibonacci se tomó 51516 ms. 
    
    
    ![WhatsApp Image 2020-04-02 at 10 34 34 AM](https://user-images.githubusercontent.com/48154086/78269951-d8a58580-74cf-11ea-928a-296c67bab150.jpeg)
  
  
    * Para calcular 1010000 número de la secuencia de Fibonacci se tomó 52688 ms. 
    
    
   ![WhatsApp Image 2020-04-02 at 10 35 53 AM](https://user-images.githubusercontent.com/48154086/78270257-3508a500-74d0-11ea-9718-abf5db078865.jpeg)
   
   
    * Para calcular 1020000 número de la secuencia de Fibonacci se tomó 93544 ms. 
    
    
   ![WhatsApp Image 2020-04-02 at 9 54 20 AM (1)](https://user-images.githubusercontent.com/48154086/78269575-65037880-74cf-11ea-9049-22362ccb1735.jpeg)
   
   
    * Para calcular 1030000 número de la secuencia de Fibonacci se tomó 95674 ms.
    
    
    ![WhatsApp Image 2020-04-02 at 9 57 48 AM](https://user-images.githubusercontent.com/48154086/78269579-659c0f00-74cf-11ea-80ef-507dc999c241.jpeg)
    
    
    * Para calcular 1040000 número de la secuencia de Fibonacci se tomó 97926 ms. 
    
    
   ![WhatsApp Image 2020-04-02 at 9 59 43 AM](https://user-images.githubusercontent.com/48154086/78269744-9bd98e80-74cf-11ea-8262-400a0f194b8a.jpeg)
   
   
    * Para calcular 1050000 número de la secuencia de Fibonacci se tomó 101225 ms. 
    
    
    ![WhatsApp Image 2020-04-02 at 10 01 57 AM](https://user-images.githubusercontent.com/48154086/78269755-9da35200-74cf-11ea-9e79-cb80eed8ba70.jpeg)
    
    
    * Para calcular 1060000 número de la secuencia de Fibonacci se tomó 100844 ms. 
    
    
   ![WhatsApp Image 2020-04-02 at 10 04 41 AM](https://user-images.githubusercontent.com/48154086/78269925-d2170e00-74cf-11ea-8c1f-9ce7c1326554.jpeg)
    
    
    * Para calcular 1070000 número de la secuencia de Fibonacci se tomó 102394 ms. 
    
    
    ![WhatsApp Image 2020-04-02 at 10 06 39 AM](https://user-images.githubusercontent.com/48154086/78269927-d2afa480-74cf-11ea-8282-479cde326b5d.jpeg)
    
    
    * Para calcular 1080000 número de la secuencia de Fibonacci se tomó 152100 ms. 
    
    
    ![WhatsApp Image 2020-04-02 at 10 09 44 AM](https://user-images.githubusercontent.com/48154086/78269935-d4796800-74cf-11ea-8d1b-9bcc4d803e67.jpeg)
    
    
    * Para calcular 1090000 número de la secuencia de Fibonacci se tomó 107032 ms. 
    
    
    ![WhatsApp Image 2020-04-02 at 10 13 01 AM](https://user-images.githubusercontent.com/48154086/78269938-d6432b80-74cf-11ea-836e-1eeaf613a51e.jpeg)
    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)


![WhatsApp Image 2020-04-02 at 9 53 15 AM](https://user-images.githubusercontent.com/48154086/78270380-5e293580-74d0-11ea-9a71-d335da6e35fa.jpeg)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

![WhatsApp Image 2020-04-02 at 11 08 30 AM](https://user-images.githubusercontent.com/48154086/78272120-b5300a00-74d2-11ea-819e-8597eaf208d9.jpeg)



10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

![WhatsApp Image 2020-04-02 at 10 59 45 AM (1)](https://user-images.githubusercontent.com/48154086/78272198-d3960580-74d2-11ea-9bb6-2908376b0623.jpeg)


11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

Paso 7. 

   * Para calcular el 1000000 número de la secuencia de Fibonacci se tomó 51516 ms. 

   ![WhatsApp Image 2020-04-02 at 10 34 34 AM (1)](https://user-images.githubusercontent.com/48154086/78273445-7307c800-74d4-11ea-9ef7-0fba31a27421.jpeg)
   
   * Para calcular 1010000 número de la secuencia de Fibonacci se tomó 52688 ms.
  
   ![WhatsApp Image 2020-04-02 at 10 35 53 AM (1)](https://user-images.githubusercontent.com/48154086/78273446-73a05e80-74d4-11ea-85ea-4e8676406cfc.jpeg)
   
   * Para calcular 1020000 número de la secuencia de Fibonacci se tomó 53544 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 37 43 AM](https://user-images.githubusercontent.com/48154086/78273450-74d18b80-74d4-11ea-8e13-f4c2f92c8854.jpeg)
   
   * Para calcular 1030000 número de la secuencia de Fibonacci se tomó 54913 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 38 52 AM](https://user-images.githubusercontent.com/48154086/78273455-7602b880-74d4-11ea-9664-45e113a2cc08.jpeg)
   
   * Para calcular 1040000 número de la secuencia de Fibonacci se tomó 55391 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 40 03 AM](https://user-images.githubusercontent.com/48154086/78273460-77cc7c00-74d4-11ea-9a33-77ce78493a4b.jpeg)
   
   * Para calcular 1050000 número de la secuencia de Fibonacci se tomó 56249 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 41 40 AM](https://user-images.githubusercontent.com/48154086/78273471-7ac76c80-74d4-11ea-85bb-d808f357117d.jpeg)
   
   * Para calcular 1060000 número de la secuencia de Fibonacci se tomó 57411 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 43 32 AM](https://user-images.githubusercontent.com/48154086/78273478-7d29c680-74d4-11ea-9bbf-4d8b5f758cd0.jpeg)
   
   * Para calcular 1070000 número de la secuencia de Fibonacci se tomó 58434 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 44 47 AM](https://user-images.githubusercontent.com/48154086/78273486-7f8c2080-74d4-11ea-8266-c102a17e2f0b.jpeg)
   
   * Para calcular 1080000 número de la secuencia de Fibonacci se tomó 59335 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 46 18 AM](https://user-images.githubusercontent.com/48154086/78273494-82871100-74d4-11ea-9a78-b354831278b0.jpeg)
   
   * Para calcular 1090000 número de la secuencia de Fibonacci se tomó 60237 ms.
   
   ![WhatsApp Image 2020-04-02 at 10 47 53 AM](https://user-images.githubusercontent.com/48154086/78273498-85820180-74d4-11ea-876f-91ef361d9107.jpeg)

Paso 8. 

   ![WhatsApp Image 2020-04-02 at 10 48 55 AM](https://user-images.githubusercontent.com/48154086/78274336-adbe3000-74d5-11ea-8507-ea9a19e60e6b.jpeg)


Paso 9. 

   ![WhatsApp Image 2020-04-02 at 10 53 39 AM](https://user-images.githubusercontent.com/48154086/78274337-adbe3000-74d5-11ea-8166-218834e101a1.jpeg)


12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

Tanto con el tamaño standard A0 que se uso en un comienzo como con el tamaño standard A6 con el cual se buscaba mejorar la funcionalidad del sistema, nos damos cuenta de que en ambos casos al momento de realizar la consulta esta consume el 100% de la cpu y en ambos casos al momento de hacer solicitudes concurrentes con postman una de ellas falla. En conclusión, consideramos que usando este modelo NO se cumple con el requerimiento no funcional de escalabilidad ya que el consumo de cpu supera el 70% y no funciona correctamente solicitudes concurrentes, aunque cabe mencionar que se redujo el tiempo de la consulta casi en la mitad pasando de 102000 ms a 55000 ms aproximadamente.


13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

   ![image](https://user-images.githubusercontent.com/44879884/78311065-6f486580-7515-11ea-9c70-755e506f2368.png)

   
2. ¿Brevemente describa para qué sirve cada recurso?

   - **Virtual network: Es el bloque de construcción fundamental para la red privada en Azure.**
   - **Storage account: Proporciona un espacio de nombres único para sus datos de Azure Storage al que se puede acceder desde cualquier lugar del mundo a través de HTTP o HTTPS.**
   - **Virtual machine: Brinda la flexibilidad de la virtualización sin tener que comprar y mantener el hardware físico que lo ejecuta** 
   - **Public IP address: Se utilizan para la comunicación con Internet, incluidos los servicios públicos de Azure.**
   - **Network security group: Contiene reglas de seguridad que permiten o niegan el tráfico de red entrante a, o el tráfico de red saliente, de varios tipos de recursos de Azure.**
   - **Network interface: Permite que una máquina virtual de Azure se comunique con Internet, Azure y recursos locales.**
   - **Disk: Almacenamiento persistente de datos**

3. 

   - ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`?
      
      Porque estaríamos cerrando la ejecución del programa, es por eso que se usa el comando `forever start FibonacciApp.js`. Este comando permite que la aplicación siga corriendo mientras la maquina esta prendida.

   - ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
      
      Porque si no abrimos el puerto 3000 que es por donde está corriendo la aplicación, desde un equipo externo a la maquina virtual creada en azure no se podría consultar desde un browser la dirección: http://13.91.82.159:3000/fibonacci/123 ya que el puerto no estaría abierto.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

   ![image](https://user-images.githubusercontent.com/44879884/78314953-344c2f00-7521-11ea-8286-ff7fd53d06e1.png)

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

Al momento en el que el cliente hace la solicitud desde el browser el consumo de CPU llega al 100% lo cual no cumple con el requerimiento no funcional de escalabilidad como se muestra a continuación: 

   ![WhatsApp Image 2020-04-02 at 9 53 15 AM](https://user-images.githubusercontent.com/48154086/78270380-5e293580-74d0-11ea-9a71-d335da6e35fa.jpeg)

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:

   ![WhatsApp Image 2020-04-02 at 9 38 21 PM](https://user-images.githubusercontent.com/48154086/78318472-69a94a80-752a-11ea-9ad4-904cedfeba1e.jpeg)

    * Tiempos de ejecución de cada petición.
    
    El promedio de los tiempos de ejecución es de 1 minuto 39,6 segundos. 
    
    * Si hubo fallos documentelos y explique.
    
    No hubo ningún fallo. 

7. ¿Cuál es la diferencia entre los tamaños `A0` y `A6` (no solo busque especificaciones de infraestructura)?

   -**A0 standard:** Tiene 1 vCPUs, 0.75GiB de RAM, 1 data disk y es ofrecida por un precio de 14.60 dólares mensuales.

   -**A6 standard:** Tiene 4 vCPUs, 28GiB de RAM, 8 data disk y es ofrecida por un precio de 365.00 dólares mensuales.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

   Aunque se mejoro el tiempo de respuesta de las solicitudes el consumo de CPU sigue excediendo el 70% y las solicitudes concurrentes no funcionan correctamente.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

   Pues el rendimiento de la máquina virtual mejora considerablemente, pero la aplicación no aprovecha todos los recursos que la maquina ofrece por esto sigue presentando una alta carga de CPU y no funciona correctamente al momento de solicitudes concurrentes. Y la desventaja más grande es el aumento de 350 dólares mensuales.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

   En el consumo de CPU bajo el tiempo en el que el consumo era excesivo pero aun así sigue superando el 70% de consumo y en el tiempo de las consultas las mantuvo todas casi en un mismo tiempo de aproximadamente entre 50000ms y 60000ms, mientras que con el primer tamaño los tiempos estuvieron entre 50000ms y 120000ms.  

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

El comportamiento del sistema no es porcentualmente mejor, en algunas de las solicitudes concurrentes que se envian este no es capaz de resolverlas.  

   ![WhatsApp Image 2020-04-02 at 9 46 20 PM](https://user-images.githubusercontent.com/48154086/78319008-cce7ac80-752b-11ea-863f-a0096ff2164a.jpeg)
   ![WhatsApp Image 2020-04-02 at 9 58 12 PM](https://user-images.githubusercontent.com/48154086/78319650-2ef4e180-752d-11ea-8aac-81210257cb8b.jpeg)
   ![WhatsApp Image 2020-04-02 at 9 58 25 PM](https://user-images.githubusercontent.com/48154086/78319654-2f8d7800-752d-11ea-989d-5a26f3d3d9ff.jpeg)
   
   


### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.
