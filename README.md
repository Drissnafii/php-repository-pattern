# Repository Design Pattern: Why and How to Use It

The **Repository Design Pattern** is a software design pattern that acts as a middle layer between your business logic and data access logic. It centralizes database operations, making your code cleaner, more maintainable, and easier to test.

---

## 🧩 What is the Repository Pattern?

The Repository Pattern is like a **librarian** for your application:  
- **You (Controller)**: Ask the librarian for a book (data).  
- **Librarian (Repository)**: Knows where the books are stored and how to fetch them.  
- **Bookshelf (Database)**: The actual storage system.  

You don’t care *how* the librarian retrieves the book—you just get the book. Similarly, your application doesn’t need to know *how* data is fetched from the database.

---

## ❌ Bad Example: Without Repository Pattern

### Problem: Mixing Database Logic in Controllers
In this example, database logic is directly embedded in the controller. This leads to **tight coupling**, **duplicated code**, and **hard-to-maintain** systems.

**File**: `app/controllers/UserController.php`
```php
class UserController {
    private $db;

    public function __construct() {
        $this->db = new PDO("pgsql:host=localhost;dbname=myapp", "user", "password");
    }

    // Fetch a user by ID
    public function showProfile($userId) {
        $stmt = $this->db->prepare("SELECT * FROM users WHERE id = :id");
        $stmt->execute([':id' => $userId]);
        $user = $stmt->fetch();

        // Display user data
        echo "User: {$user['name']}, Email: {$user['email']}";
    }

    // Create a new user
    public function createUser($name, $email) {
        $stmt = $this->db->prepare("INSERT INTO users (name, email) VALUES (:name, :email)");
        $stmt->execute([':name' => $name, ':email' => $email]);
    }
}
```

### Issues with This Approach:
1. **Tight Coupling**: Controllers are tightly coupled to the database.  
2. **Duplicated Code**: If another controller needs to fetch users, the same SQL query is duplicated.  
3. **Hard to Test**: You can’t test the controller without a real database.  
4. **Security Risks**: SQL queries scattered everywhere increase the risk of SQL injection.  

---

## ✅ Good Example: With Repository Pattern

### Solution: Centralize Database Logic in a Repository
The Repository Pattern separates database logic from business logic, making your code **cleaner**, **testable**, and **reusable**.

---

### Step 1: Create the **Model**
**File**: `app/models/User.php`  
Models are plain PHP objects that represent your data.  
```php
class User {
    private $id;
    private $name;
    private $email;

    public function __construct($id, $name, $email) {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
    }

    // Getters
    public function getId() { return $this->id; }
    public function getName() { return $this->name; }
    public function getEmail() { return $this->email; }
}
```

---

### Step 2: Create the **Repository**
**File**: `app/repositories/UserRepository.php`  
The repository handles all database operations for the `User` model.  
```php
require_once __DIR__ . '/../core/Database.php';

class UserRepository {
    private $db;

    public function __construct() {
        $this->db = Database::getInstance()->getConnection();
    }

    // Fetch a user by ID
    public function findById($id) {
        $stmt = $this->db->prepare("SELECT * FROM users WHERE id = :id");
        $stmt->execute([':id' => $id]);
        $data = $stmt->fetch();

        return new User($data['id'], $data['name'], $data['email']);
    }

    // Save a user
    public function save(User $user) {
        $stmt = $this->db->prepare("INSERT INTO users (name, email) VALUES (:name, :email)");
        $stmt->execute([
            ':name' => $user->getName(),
            ':email' => $user->getEmail()
        ]);
    }
}
```

---

### Step 3: Use the Repository in a **Controller**
**File**: `app/controllers/UserController.php`  
Controllers now use the repository to interact with the database.  
```php
require_once __DIR__ . '/../repositories/UserRepository.php';

class UserController {
    private $userRepository;

    public function __construct() {
        $this->userRepository = new UserRepository();
    }

    // Fetch a user by ID
    public function showProfile($userId) {
        $user = $this->userRepository->findById($userId);
        echo "User: {$user->getName()}, Email: {$user->getEmail()}";
    }

    // Create a new user
    public function createUser($name, $email) {
        $user = new User(null, $name, $email);
        $this->userRepository->save($user);
    }
}
```

---

## 🌟 Benefits of the Repository Pattern

1. **Separation of Concerns**:  
   - Models handle business logic.  
   - Repositories handle database logic.  
   - Controllers handle HTTP logic.  

2. **Testability**:  
   - Mock repositories to test controllers without a real database.  

3. **Reusability**:  
   - Use the same repository across multiple controllers.  

4. **Security**:  
   - Centralized SQL queries make it easier to prevent SQL injection.  

5. **Maintainability**:  
   - Changes to database logic are confined to the repository.  

---

## 🔄 Workflow Analogy

1. **You (Controller)**: "Hey Repository, get me user #123!"  
2. **Repository**: Runs `SELECT * FROM users WHERE id = 123` and returns a `User` object.  
3. **You**: Display the user’s data in a view.  

No need to worry about *how* the data is fetched—just focus on what to do with it!

---

## 🚀 When to Use the Repository Pattern

- Your application has complex database queries.  
- You want to decouple your business logic from data access logic.  
- You need to support multiple data sources (e.g., PostgreSQL, MySQL, APIs).  
- You want to write unit tests for your application.  

---
---
---
---
---
---
---
---
---

Lorsque l'utilisateur clique sur le lien **"Profil"** dans la liste des utilisateurs, voici ce qui se passe étape par étape :

---

### 1. **Lien cliqué dans `user_list.php`**

Dans la vue `user_list.php`, chaque utilisateur a un lien vers son profil :

```php
<a href="index.php?action=profile&id=<?php echo $user->id; ?>">👤 Profil</a>
```

- **Exemple** : Si l'utilisateur a l'ID 1, le lien sera :
  ```
  index.php?action=profile&id=1
  ```

---

### 2. **Traitement dans `index.php`**

Le fichier `index.php` reçoit la requête et appelle le contrôleur approprié :

```php
// index.php
$action = $_GET['action'] ?? 'list';
$id = $_GET['id'] ?? null;

switch ($action) {
    case 'profile':
        $profileController->showProfile($_GET['id']);
        break;
    // ... autres cas
}
```

- **Action** : `profile`
- **ID** : `1` (ou l'ID de l'utilisateur cliqué)

---

### 3. **Appel de la méthode `showProfile` dans `ProfileController`**

Le contrôleur `ProfileController` récupère l'utilisateur via le repository et affiche la vue `profile.php` :

```php
// controllers/ProfileController.php
public function showProfile($userId)
{
    $user = $this->userRepository->find($userId);
    if ($user) {
        include 'views/profile.php';
    } else {
        echo "Utilisateur non trouvé.";
    }
}
```

- **Étapes** :
  1. Le repository est utilisé pour récupérer l'utilisateur avec l'ID donné.
  2. Si l'utilisateur existe, la vue `profile.php` est chargée.
  3. Si l'utilisateur n'existe pas, un message d'erreur est affiché.

---

### 4. **Affichage de la vue `profile.php`**

La vue `profile.php` affiche les détails de l'utilisateur :

```php
<!-- views/profile.php -->
<!DOCTYPE html>
<html>
<head>
    <title>Profil de <?php echo $user->name; ?></title>
</head>
<body>
    <h1>Profil</h1>
    <p>ID : <?php echo $user->id; ?></p>
    <p>Nom : <?php echo $user->name; ?></p>
    <p>Email : <?php echo $user->email; ?></p>
</body>
</html>
```

- **Résultat** : Une page HTML avec les informations de l'utilisateur (ID, nom, email).

---

### 5. **Résultat dans le navigateur**

Après avoir cliqué sur le lien "Profil" pour l'utilisateur avec l'ID 1, tu verras une page comme ceci :

```
Profil
ID : 1
Nom : driss
Email : driss@gmail.com
```

---

### 6. **Gestion des erreurs**

Si l'utilisateur n'existe pas (ex: ID invalide), un message d'erreur est affiché :

```
Utilisateur non trouvé.
```

---

### Exemple complet

#### URL cliquée :
```
index.php?action=profile&id=1
```

#### Flux :
1. Le lien redirige vers `index.php` avec `action=profile` et `id=1`.
2. `index.php` appelle `ProfileController->showProfile(1)`.
3. Le repository récupère l'utilisateur avec l'ID 1.
4. La vue `profile.php` affiche les détails de l'utilisateur.

---

### Améliorations possibles

1. **Ajouter un bouton "Retour"** :
   Dans `profile.php`, ajoute un lien pour revenir à la liste des utilisateurs :

   ```php
   <a href="index.php">Retour à la liste</a>
   ```

2. **Ajouter des informations supplémentaires** :
   Si tu as d'autres données (ex: adresse, téléphone), tu peux les afficher dans le profil.

3. **Gestion des permissions** :
   Vérifie si l'utilisateur connecté a le droit de voir ce profil.

---

### Conclusion

Après avoir cliqué sur "Profil", l'application :
1. Récupère l'utilisateur via le repository.
2. Affiche ses informations dans une vue dédiée.
---
---
---
---
---
---
---
---
---
---
---
---
---
---
---
---
---
### **Récapitulatif Final : Le Repository Design Pattern**

Le **Repository Design Pattern** est une solution élégante pour gérer l’accès aux données tout en maintenant une séparation claire entre les couches de votre application. **Pourquoi est-il né ?** Parce que les applications modernes ont besoin de flexibilité, de maintenabilité, et de testabilité, surtout quand il s’agit d’interagir avec des bases de données, des APIs, ou d’autres sources de données. Sans ce pattern, le code devient vite désorganisé, avec des requêtes SQL éparpillées, une logique métier mélangée à la persistance, et une difficulté à changer de technologie sans tout réécrire.

**Comment résout-il ces problèmes ?**  
- **Découplage** : En isolant l’accès aux données dans des classes dédiées (*repositories*), la logique métier (contrôleurs) ne dépend plus des détails techniques (SQL, MongoDB, etc.).  
- **Abstraction** : Une **interface** (ex: `UserRepositoryInterface`) définit un *contrat* que toutes les implémentations doivent respecter. Que vous utilisiez MySQL, une API REST, ou un mock pour les tests, les contrôleurs appellent les mêmes méthodes (`find()`, `save()`, etc.).  
- **Réutilisabilité** : Un même repository peut être injecté dans **plusieurs contrôleurs** (ex: `UserController`, `AdminController`), chacun l’utilisant selon ses besoins (affichage basique vs. logique admin enrichie).  
- **Testabilité** : Vous pouvez simuler des données avec des repositories *factices* (`FakeUserRepository`) pour tester la logique métier sans base de données réelle.  

**Exemple concret** :  
- Un `UserRepository` (MySQL) et un `AdminUserRepository` (avec tri et métadonnées supplémentaires) implémentent la même interface.  
- Les contrôleurs `UserController` et `AdminController` utilisent ces repositories sans savoir comment les données sont récupérées.  
- Résultat : Vous pouvez basculer de MySQL à MongoDB, ou ajouter une couche de cache, **sans toucher aux contrôleurs**.  

**Bénéfices clés** :  
- 🛠 **Maintenance facilitée** : Les changements se font en un seul endroit.  
- 🧪 **Tests simplifiés** : Mocks et isolation des composants.  
- 🚀 **Évolutivité** : Adaptez-vous à de nouvelles technologies sans casser l’existant.  
- 🧠 **Clarté du code** : Séparation stricte entre métier, données, et affichage (MVC).  

En adoptant ce pattern, vous construisez des applications **robustes**, **flexibles**, et **prêtes à évoluer**. Que ce soit pour une petite application ou un système complexe, le Repository Design Pattern est un allié indispensable pour un code propre et professionnel.  

---

**Merci pour votre attention !** 🙏  
N’hésitez pas à poser des questions ou à explorer des cas d’utilisation concrets pour approfondir votre maîtrise de ce pattern essentiel. 🚀
