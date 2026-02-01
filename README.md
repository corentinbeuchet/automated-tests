# 🧪 Exercice 3 – Intégration de tests automatisés dans un pipeline CI/CD
*(Spring Boot + Gradle)*

---

## 📚 Contexte
Cet exercice fait suite aux précédents où :
- un dépôt GitHub a été créé et sécurisé,
- un pipeline CI/CD a été mis en place,
- des règles de protection de branche ont été configurées.

Vous allez **générer un projet Spring Boot avec Spring Initializr**, y ajouter des **tests automatisés**, puis les intégrer dans un **pipeline CI/CD**.

---

## 🎯 Objectifs pédagogiques
À la fin de cet exercice, vous serez capable de :
- Générer un projet Spring Boot avec Spring Initializr
- Utiliser **Gradle** comme outil de build
- Configurer le projet pour **Java 25**
- Mettre en place des tests unitaires et d’intégration
- Intégrer l’exécution des tests dans un pipeline CI/CD
- Comprendre comment la CI bloque un merge lorsque les tests échouent

---

# 🧩 PARTIE 1 – Génération du projet Spring Boot

## 1️⃣ Configuration Spring Initializr
Rendez-vous sur :  
👉 https://start.spring.io

Utilisez la configuration suivante :

- **Project** : Gradle
- **Language** : Java
- **Spring Boot** : **4.1.0 (M1)**
- **Group** : `ort.lyon`
- **Artifact** : `demo`
- **Name** : `demo`
- **Packaging** : Jar
- **Configuration** : YAML
- **Java** : **25**
- **Dependencies** :
    - Spring Web

Cliquez sur **Generate** et téléchargez le projet.

---

## 2️⃣ Pousser le projet sur GitHub
Décompressez l’archive puis exécutez :

```bash
git init
git add .
git commit -m "Initialisation du projet Spring Boot (Gradle)"
git branch -M main
git push origin main
```

Vérifiez que le projet fonctionne localement :

```bash
./gradlew test
```

---

## 3️⃣ Ajouter les règles pour protéger la branch main
Comme pour l'exercice précédent, protégez la branche main avec les *branch protection rule*


# 🧩 PARTIE 2 – Création d’une API REST simple

## 3️⃣ Création d’un contrôleur REST
Créez la classe suivante :

```java
package ort.lyon.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello CI/CD";
    }
}
```

Lancez l’application :

```bash
./gradlew bootRun
```

Testez dans votre navigateur :  
👉 http://localhost:8080/hello

---

# 🧩 PARTIE 3 – Ajout de tests automatisés

## 4️⃣ Test unitaire (JUnit)
Créez le test suivant :

```java
package ort.lyon.demo;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class HelloControllerTest {

    @Test
    void leMessageHelloDoitEtreCorrect() {
        String message = "Hello CI/CD";
        assertEquals("Hello CI/CD", message);
    }
}
```

Exécutez les tests :

```bash
./gradlew test
```

---

## 5️⃣ Test d’intégration (Spring Boot)
Jetez un coup d'oeil au test d’intégration mis par défaut:

```java
package ort.lyon.demo;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class DemoApplicationTests {

	@Test
	void contextLoads() {
	}
}
```

📌 Ce test vérifie que le contexte Spring démarre sans erreur.

---

# 🧩 PARTIE 4 – Intégration des tests dans la CI/CD

## 6️⃣ Workflow GitHub Actions
Créez le fichier suivant :

```
.github/workflows/ci.yml
```

```yaml
name: CI

on:
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '25'
      - name: Lancer les tests
        run: ./gradlew test
```

---

# 🧩 PARTIE 5 – Validation du comportement de la CI

## 7️⃣ Déclenchement du pipeline
1. Créez une branche de fonctionnalité :
```bash
git switch -c feat/tests
```
2. Poussez vos modifications
3. Ouvrez une *Pull Request* vers `main`

📌 Résultat attendu :
- La CI se lance automatiquement
- Le merge est bloqué tant que les tests ne passent pas

---

## 8️⃣ Forcer un échec de test
Modifiez volontairement un test pour le faire échouer.

Résultat attendu :
- La CI passe au rouge
- Le merge est bloqué

Corrigez ensuite le test et vérifiez que la CI redevient verte.

---

# ❓ Questions de réflexion
1. Pourquoi les tests automatisés sont-ils essentiels dans un pipeline CI/CD ?
2. Quelle est la différence entre tests unitaires et tests d’intégration ?
3. Pourquoi la CI ne suffit-elle pas sans revue de code ?
4. Que se passerait-il si les tests étaient exécutés uniquement manuellement ?

---

# 🏁 Conclusion
Cet exercice illustre un workflow CI/CD professionnel basé sur :
- Spring Boot
- Gradle
- Tests automatisés
- GitHub Actions

Il constitue une base standard utilisée dans de nombreux projets DevOps modernes.
