# 🧪 Exercice 2 – Intégration de tests automatisés dans un pipeline CI/CD
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
- Mettre en place des tests unitaires et d'intégration
- Intégrer l'exécution des tests dans un pipeline CI/CD
- Comprendre comment la CI bloque un merge lorsque les tests échouent

---

## ✅ Prérequis
- **JDK 25** installé sur votre poste (la version LTS actuelle, par exemple Eclipse Temurin 25). Vérifiez-le avec `java -version`.
- Git et un compte GitHub
- Un IDE Java (IntelliJ IDEA, VS Code…)

> 💻 **Windows** : dans ce document, remplacez `./gradlew` par `.\gradlew.bat`.

---

# 🧩 PARTIE 1 – Génération du projet Spring Boot

## 1️⃣ Configuration Spring Initializr
Rendez-vous sur :
👉 https://start.spring.io

Utilisez la configuration suivante :

- **Project** : Gradle - Groovy
- **Language** : Java
- **Spring Boot** : la dernière version **stable** (sélectionnée par défaut, **sans** mention SNAPSHOT, M ou RC)
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

## 2️⃣ Vérifier le projet puis le pousser sur GitHub
Décompressez l'archive, ouvrez un terminal dans le dossier du projet et vérifiez d'abord que tout fonctionne localement :

```bash
./gradlew test
```

Sur GitHub, créez un nouveau dépôt **public** `automated-tests`, **vide** : ne cochez ni README, ni .gitignore, ni licence, sinon votre premier push sera refusé.

Puis :

```bash
git init
git add .
git commit -m "chore: init Spring Boot project (Gradle)"
git branch -M main
git remote add origin https://github.com/<votre-compte>/automated-tests.git
git push -u origin main
```

---

## 3️⃣ Protéger la branche main
Comme dans l'exercice précédent, ajoutez une règle sur `main` (**Settings → Branches → Add classic branch protection rule**) :
- ✅ **Require a pull request before merging**, puis **décochez Require approvals** (cochée par défaut) : vous êtes seul, personne ne pourrait approuver votre PR
- ✅ **Do not allow bypassing the above settings**

> La règle « CI obligatoire » sera ajoutée à l'étape 8 : GitHub ne peut la proposer qu'une fois la CI lancée au moins une fois.

---

# 🧩 PARTIE 2 – Création d'une API REST simple

## 4️⃣ Création d'un contrôleur REST
Créez la classe suivante dans `src/main/java/ort/lyon/demo/` :

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

Lancez l'application :

```bash
./gradlew bootRun
```

Testez dans votre navigateur :
👉 http://localhost:8080/hello

Arrêtez ensuite l'application (`Ctrl+C`).

---

# 🧩 PARTIE 3 – Ajout de tests automatisés

## 5️⃣ Test unitaire (JUnit)
Créez le test suivant dans `src/test/java/ort/lyon/demo/` :

```java
package ort.lyon.demo;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class HelloControllerTest {

    @Test
    void leMessageHelloDoitEtreCorrect() {
        HelloController controller = new HelloController();

        String message = controller.hello();

        assertEquals("Hello CI/CD", message);
    }
}
```

📌 Ce test appelle vraiment le contrôleur : si quelqu'un modifie le message, il échouera.

Exécutez les tests :

```bash
./gradlew test
```

---

## 6️⃣ Test d'intégration (Spring Boot)
Jetez un coup d'œil au test d'intégration généré par défaut :

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

## 7️⃣ Workflow GitHub Actions
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
      - uses: actions/checkout@v7
      - uses: actions/setup-java@v5
        with:
          distribution: 'temurin'
          java-version: '25'
      - name: Lancer les tests
        run: ./gradlew test
```

---

# 🧩 PARTIE 5 – Validation du comportement de la CI

## 8️⃣ Déclenchement du pipeline
1. Créez une branche de fonctionnalité, puis committez et poussez tout votre travail :
```bash
git switch -c feat/tests
git add .
git commit -m "test: add HelloController, unit test and CI workflow"
git push -u origin feat/tests
```
2. Ouvrez une *Pull Request* de `feat/tests` vers `main`.

📌 Résultat attendu :
- La CI se lance automatiquement **mais est en erreur**. Trouvez une solution en observant les logs (onglet **Checks** de la PR).
- Une fois la CI verte, retournez dans la règle de protection de `main` et activez **Require status checks to pass before merging** avec le check **`build`**. Désormais, le merge est bloqué tant que les tests ne passent pas.

> 💡 **Vous êtes sous macOS ou Linux et la CI est verte du premier coup ?** C'est normal : l'erreur n'apparaît que lorsque le projet a été committé depuis Windows. Demandez à un camarade sous Windows de vous montrer ses logs, puis passez à l'étape 9.

<details>
<summary>Indice (à n'ouvrir qu'après avoir lu les logs)</summary>

Le log indique `./gradlew: Permission denied`. Windows ne conserve pas le droit d'exécution des fichiers : le script `gradlew` est arrivé sur GitHub sans ce droit.
Deux solutions :
- corriger le fichier dans Git (recommandé) : `git update-index --chmod=+x gradlew`, puis commit et push ;
- ou ajouter une étape `chmod +x ./gradlew` dans le workflow, avant `./gradlew test`.
</details>

---

## 9️⃣ Provoquer une régression
Dans `HelloController`, changez le message renvoyé (par exemple `"Hello"`), committez et poussez sur votre branche.

Résultat attendu :
- Le test unitaire échoue et la CI passe au **rouge**
- Le **merge est bloqué**

Remettez le bon message et vérifiez que la CI redevient verte. Vous pouvez alors merger.

---

# ❓ Questions de réflexion
1. Pourquoi les tests automatisés sont-ils essentiels dans un pipeline CI/CD ?
2. Quelle est la différence entre tests unitaires et tests d'intégration ?
3. Pourquoi la CI ne suffit-elle pas sans revue de code ?
4. Que se passerait-il si les tests étaient exécutés uniquement manuellement ?
5. Si l'on supprime le test unitaire, la CI reste verte. Qu'est-ce que cela dit de la confiance qu'on peut accorder à une CI verte ?

---

# 🏁 Conclusion
Cet exercice illustre un workflow CI/CD professionnel basé sur :
- Spring Boot
- Gradle
- Tests automatisés
- GitHub Actions

Il constitue une base standard utilisée dans de nombreux projets DevOps modernes.
