# 🧪 Exercice 3 – Tests automatisés, livrable et qualité dans un pipeline CI/CD

> 🎯 **Priorités** : tout cet exercice est l'essentiel (tests, livrable, qualité, performance), ce que vous devrez savoir refaire seul à l'évaluation finale. Seul le bonus de la fin est pour aller plus loin.

*(Spring Boot + Gradle)*

---

## 📚 Contexte
Cet exercice fait suite à l'exercice 2, où vous avez :
- créé et sécurisé un dépôt GitHub,
- mis en place une première CI,
- protégé la branche `main` (PR obligatoire, CI bloquante, revue).

La CI de l'exercice 2 ne « testait » rien : un simple `echo`. Ici, vous allez **générer un vrai projet Spring Boot**, écrire des **tests automatisés**, puis faire produire à la CI un **livrable**, un **contrôle qualité** et un **test de performance**.

---

## 🎯 Ce que vous devez comprendre et savoir faire
À la fin de cet exercice, vous devez être capable de :
- **Générer et lancer** un projet Spring Boot avec Gradle et Java 25 (LTS).
- **Écrire un test unitaire qui a du sens** : il appelle le vrai code, et il échoue si le comportement change.
- **Faire la différence** entre un test unitaire (une classe isolée) et un test d'intégration (l'application démarrée).
- **Lire les logs d'une CI en échec** et en trouver la cause, sans deviner.
- **Expliquer ce qu'est un livrable** (le `.jar`) et pourquoi on le construit **une seule fois**, dans la CI, plutôt que sur le poste d'un développeur.
- **Expliquer pourquoi un contrôle de style automatique** (Checkstyle) évite des débats en revue de code.
- **Expliquer ce qu'est un test non fonctionnel** : l'application répond juste, mais répond-elle assez vite, sous charge ?
- **Dire ce que garantit, et ne garantit pas, une CI verte.**

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
    runs-on: ubuntu-26.04
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-java@v6
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

# 🧩 PARTIE 6 – Produire un livrable

Jusqu'ici, la CI dit seulement « les tests passent ». Mais ce qu'on déploie, c'est un **fichier** : le `.jar` de l'application. On veut qu'il soit construit **par la CI**, toujours de la même façon, et récupérable.

## 🔟 Donner un nom fixe au jar
À la fin de `build.gradle`, ajoutez :

```groovy
tasks.named('bootJar') {
    archiveFileName = 'app.jar'
}
```

Vérifiez en local :

```bash
./gradlew build
java -jar build/libs/app.jar
```

> `./gradlew build` fait plus que `test` : il compile, lance **toutes** les vérifications (`check`) et fabrique le jar.

## 1️⃣1️⃣ Publier le livrable depuis la CI
Sur une nouvelle branche (`feat/livrable`), remplacez la dernière étape du job `build` de `ci.yml` par :

```yaml
      - name: Construire et vérifier
        run: ./gradlew build
      - name: Publier le livrable
        uses: actions/upload-artifact@v7
        with:
          name: app
          path: build/libs/app.jar
      - name: Publier les rapports (même en cas d'échec)
        if: always()
        uses: actions/upload-artifact@v7
        with:
          name: rapports
          path: build/reports/
```

Poussez, ouvrez une PR, puis ouvrez le run dans l'onglet **Actions** : en bas de la page, la section **Artifacts** contient `app` et `rapports`. Téléchargez `rapports` et ouvrez `tests/test/index.html`.

📌 Le jar que vous téléchargez est **exactement** celui qui a été testé. C'est lui, et pas un jar reconstruit sur un poste, qu'on déploiera.

Une fois la CI verte, mergez la PR.

---

# 🧩 PARTIE 7 – Qualité du code (Checkstyle)

Des règles de style vérifiées **par une machine** : plus besoin d'en débattre en revue de code, la revue peut se concentrer sur la logique.

## 1️⃣2️⃣ Activer Checkstyle
Sur une nouvelle branche (`feat/qualite`), dans le bloc `plugins` de `build.gradle`, ajoutez `id 'checkstyle'`, puis ajoutez à la fin du fichier :

```groovy
checkstyle {
    toolVersion = '14.1.0'   // dernière version : https://checkstyle.org
    maxWarnings = 0
}
```

Créez le fichier de règles `config/checkstyle/checkstyle.xml` :

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">
<module name="Checker">
  <module name="LineLength">
    <property name="max" value="120"/>
  </module>
  <module name="TreeWalker">
    <module name="AvoidStarImport"/>
    <module name="UnusedImports"/>
    <module name="RedundantImport"/>
    <module name="NeedBraces"/>
    <module name="EmptyCatchBlock"/>
    <module name="EqualsHashCode"/>
    <module name="SimplifyBooleanExpression"/>
  </module>
</module>
```

## 1️⃣3️⃣ Constater et corriger
Lancez :

```bash
./gradlew check
```

📌 Résultat attendu : **échec**. Lisez le message : il indique le fichier, la ligne et la règle enfreinte. Le rapport détaillé est dans `build/reports/checkstyle/`.

Corrigez le code (pas la règle !), relancez `./gradlew check` jusqu'à ce qu'il passe, puis poussez et ouvrez une PR : la CI (`./gradlew build`) applique maintenant les mêmes règles. Mergez une fois la CI verte.

<details>
<summary>Indice</summary>

C'est le test unitaire : `import static org.junit.jupiter.api.Assertions.*;` importe tout avec `*`. Importez seulement ce qui sert : `import static org.junit.jupiter.api.Assertions.assertEquals;`
</details>

---

# 🧩 PARTIE 8 – Test de performance (k6)

Les tests de la partie 3 vérifient que l'application répond **juste**. Un test **non fonctionnel** vérifie qu'elle répond **assez vite**, même quand beaucoup d'utilisateurs l'appellent en même temps.

## 1️⃣4️⃣ Le scénario de charge
Sur une nouvelle branche (`feat/perf`), créez `perf/hello.js` :

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 20,            // 20 utilisateurs virtuels en parallèle
  duration: '20s',
  thresholds: {
    http_req_failed: ['rate<0.01'],     // moins de 1 % d'erreurs
    http_req_duration: ['p(95)<200'],   // 95 % des requêtes en moins de 200 ms
  },
};

export default function () {
  const res = http.get('http://localhost:8080/hello');
  check(res, { 'statut 200': (r) => r.status === 200 });
}
```

📌 Les `thresholds` sont le critère de réussite : si l'un n'est pas respecté, k6 échoue, et la CI aussi.

## 1️⃣5️⃣ Le job de performance
Ajoutez ce second job à `ci.yml`, au même niveau que `build` :

```yaml
  performance:
    needs: build
    runs-on: ubuntu-26.04
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-java@v6
        with:
          distribution: 'temurin'
          java-version: '25'
      - name: Récupérer le livrable construit par le job build
        uses: actions/download-artifact@v8
        with:
          name: app
          path: build/libs
      - name: Démarrer l'application
        run: |
          java -jar build/libs/app.jar &
          for i in $(seq 1 30); do
            curl -fs http://localhost:8080/hello && exit 0
            sleep 2
          done
          echo "L'application n'a pas démarré" && exit 1
      - uses: grafana/setup-k6-action@v1
      - uses: grafana/run-k6-action@v1
        with:
          path: perf/hello.js
```

Poussez et ouvrez une PR. Dans les logs du job `performance`, repérez la durée `p(95)` et le taux d'erreurs.

> 📌 `needs: build` : le test de charge ne tourne que si le build est vert, et il teste **le même jar** que celui publié à la partie 6.

## 1️⃣6️⃣ Faire échouer le test de performance
Rendez le seuil irréaliste (`p(95)<1`), poussez : le job `performance` passe au **rouge**. Remettez `200`, puis mergez.

Bonus : ajoutez `performance` aux checks obligatoires de `main`.

---

# ❓ Questions de réflexion
1. Pourquoi les tests automatisés sont-ils essentiels dans un pipeline CI/CD ?
2. Quelle est la différence entre tests unitaires et tests d'intégration ?
3. Pourquoi la CI ne suffit-elle pas sans revue de code ?
4. Que se passerait-il si les tests étaient exécutés uniquement manuellement ?
5. Si l'on supprime le test unitaire, la CI reste verte. Qu'est-ce que cela dit de la confiance qu'on peut accorder à une CI verte ?
6. Pourquoi publier le jar depuis la CI plutôt que de le construire sur son poste au moment de déployer ?
7. Le test de performance tourne sur une machine GitHub partagée. Quelles limites cela pose-t-il pour interpréter ses résultats ?

---

# 🏁 Conclusion
Votre pipeline fait maintenant ce qu'on attend d'une vraie CI :

**tests → contrôle qualité → livrable → test de performance**

À l'exercice 4, vous allez **déployer** ce livrable : dans une image Docker, puis vers plusieurs environnements avec Ansible.
