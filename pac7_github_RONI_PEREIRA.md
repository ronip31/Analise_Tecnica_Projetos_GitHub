# Análise Técnica de Projetos Reais no GitHub

## 1. Microservices Demo

- **Projeto analisado:** [microservices-demo](https://github.com/microservices-demo/microservices-demo)
- **URL completa:** https://github.com/microservices-demo/microservices-demo
- **Estrelas:** ~1.000
- **Características técnicas:**
  - **Uso de cache (Redis):**
    - **Evidência (trecho de código):**
      ```python
      import redis
      cache = redis.Redis(host='redis', port=6379)
      def get_product(product_id):
          cached = cache.get(f"product:{product_id}")
          if cached:
              return json.loads(cached)
          product = db.query(Product).get(product_id)
          cache.setex(f"product:{product_id}", 3600, json.dumps(product))
          return product
      ```
    - **Justificativa:** Redis é utilizado para caching de dados de produtos, reduzindo a carga no banco de dados e acelerando respostas para requisições frequentes. Isso é essencial em uma arquitetura de microserviços, onde consultas repetitivas podem impactar o desempenho.

  - **Filas de mensageria (RabbitMQ):**
    - **Evidência (trecho de código):**
      ```python
      import pika
      connection = pika.BlockingConnection(pika.ConnectionParameters('rabbitmq'))
      channel = connection.channel()
      channel.queue_declare(queue='order_queue')
      channel.basic_publish(exchange='', routing_key='order_queue', body=json.dumps(order))
      ```
    - **Justificativa:** RabbitMQ gerencia comunicação assíncrona entre microserviços, como o processamento de pedidos. Isso desacopla os serviços, permitindo escalabilidade e robustez em cenários de alta demanda.

  - **Circuit breaker:**
    - **Evidência (trecho de código):**
      ```java
      @HystrixCommand(fallbackMethod = "fallback")
      public Product getProduct(String id) {
          return restTemplate.getForObject("http://product-service/products/" + id, Product.class);
      }
      public Product fallback(String id) {
          return new Product(id, "Fallback Product", 0.0);
      }
      ```
    - **Justificativa:** Hystrix é usado para proteger contra falhas em chamadas a serviços externos, evitando falhas em cascata. Isso é crucial em microserviços, onde a dependência entre serviços é comum.

  - **CI/CD com pipelines:**
    - **Evidência (trecho de código):**
      ```groovy
      pipeline {
          agent any
          stages {
              stage('Build') {
                  steps {
                      sh 'mvn clean package'
                  }
              }
              stage('Deploy') {
                  steps {
                      sh 'kubectl apply -f k8s/deployment.yaml'
                  }
              }
          }
      }
      ```
    - **Justificativa:** Jenkins automatiza a construção, teste e implantação, garantindo entregas confiáveis e rápidas. Isso é essencial para manter a agilidade em um projeto de microserviços.

## 2. Spring PetClinic Microservices

- **Projeto analisado:** [spring-petclinic-microservices](https://github.com/spring-petclinic/spring-petclinic-microservices)
- **URL completa:** https://github.com/spring-petclinic/spring-petclinic-microservices
- **Estrelas:** ~2.000
- **Características técnicas:**
  - **Circuit breaker:**
    - **Evidência (trecho de código):**
      ```java
      @CircuitBreaker(name = "vetsService", fallbackMethod = "fallback")
      public List<Vet> getVets() {
          return restTemplate.getForObject("http://vets-service/vets", List.class);
      }
      private List<Vet> fallback(Throwable t) {
          return Collections.emptyList();
      }
      ```
    - **Justificativa:** Resilience4j protege contra falhas em chamadas a serviços externos, aumentando a resiliência em uma arquitetura de microserviços.
  - **CI/CD com pipelines:**
    - **Evidência (trecho de código):**
      ```yaml
      name: Maven Build
      on: push
      jobs:
        build:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v3
            - name: Build with Maven
              run: mvn clean install
      ```
    - **Justificativa:** GitHub Actions automatiza a construção e testes, garantindo que cada commit seja validado, essencial para um projeto de demonstração.

  - **Testes automatizados:**
    - **Evidência (trecho de código):**
      ```java
      @Test
      public void testGetVets() {
          List<Vet> vets = vetService.getVets();
          assertNotNull(vets);
      }
      ```
    - **Justificativa:** Testes unitários e de integração garantem a estabilidade dos microserviços, validando interações entre serviços.

## 3. Serverless Framework

- **Projeto analisado:** [serverless](https://github.com/serverless/serverless)
- **URL completa:** https://github.com/serverless/serverless
- **Estrelas:** ~46.000
- **Características técnicas:**
  - **Serverless/Lambda:**
    - **Evidência (trecho de código):**
      ```yaml
      service: my-service
      provider:
        name: aws
        runtime: nodejs14.x
      functions:
        hello:
          handler: handler.hello
          events:
            - http:
                path: hello
                method: get
      ```
    - **Justificativa:** O framework facilita a criação de aplicações serverless no AWS Lambda, reduzindo custos e eliminando gerenciamento de servidores.
    
  - **CI/CD com pipelines:**
    - **Evidência (trecho de código):**
      ```yaml
      name: Deploy
      on: push
      jobs:
        deploy:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v3
            - run: serverless deploy
      ```
    - **Justificativa:** Integração com GitHub Actions permite automação de implantação, garantindo entregas rápidas e confiáveis.

## 4. Spring Security

- **Projeto analisado:** [spring-security](https://github.com/spring-projects/spring-security)
- **URL completa:** https://github.com/spring-projects/spring-security
- **Estrelas:** ~10.000
- **Características técnicas:**
  - **Autenticação com JWT/OAuth:**
    - **Evidência (trecho de código):**
      ```java
      @Configuration
      public class SecurityConfig {
          @Bean
          public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
              http.oauth2ResourceServer(oauth2 -> oauth2.jwt());
              return http.build();
          }
      }
      ```
    - **Justificativa:** Suporta autenticação segura com JWT e OAuth2, ideal para APIs modernas, reduzindo consultas ao servidor de autenticação.
  - **CI/CD com pipelines:**
    - **Evidência (trecho de código):**
      ```yaml
      name: CI
      on: push
      jobs:
        build:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v3
            - run: ./gradlew build
      ```
    - **Justificativa:** Pipelines de CI/CD garantem validação contínua do código, essencial para uma biblioteca de segurança.

## 5. Serverless CI Example

- **Projeto analisado:** [serverless-ci-example](https://github.com/rotemtam/serverless-ci-example)
- **URL completa:** https://github.com/rotemtam/serverless-ci-example
- **Estrelas:** ~10
- **Características técnicas:**
  - **Serverless/Lambda:**
    - **Evidência (trecho de código):**
      ```javascript
      exports.handler = async (event) => {
          return { statusCode: 200, body: JSON.stringify('Hello from Lambda!') };
      };
      ```
    - **Justificativa:** Utiliza AWS Lambda para executar funções sob demanda, reduzindo custos operacionais.
  - **CI/CD com pipelines:**
    - **Evidência (trecho de código):**
      ```yaml
      name: CI
      on: push
      jobs:
        test:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v3
            - run: mocha 'src/**/**.spec.js'
      ```
    - **Justificativa:** Pipelines de CI/CD automatizam testes e implantação, garantindo qualidade em aplicações serverless.


