### 1. Trigger de Mise à Jour de la Disponibilité des Voitures lors d'une Réservation

- **Nom du Trigger** : `Miseajour`
- **Événement** : `AFTER INSERT` sur la table `Réservations`
- **Objectif** :
    - Met à jour la table `Voitures` pour indiquer qu'une voiture réservée n'est plus disponible.

#### Code SQL :

```sql
DELIMITER //
CREATE TRIGGER Miseajour
    AFTER INSERT
    ON Réservations
    FOR EACH ROW
BEGIN
    UPDATE Voitures
    SET disponible = FALSE
    WHERE id = NEW.voiture_id;
END;//

```
---


### 2. Trigger de Vérification de l'Âge Minimum pour une Réservation

- **Nom du Trigger** : `verifage`
- **Événement** : `BEFORE INSERT` sur la table `Réservations`
- **Objectif** :
    - Vérifie que l'âge du client est supérieur ou égal à 21 ans avant d'autoriser une réservation.

#### Code SQL :

```sql
DELIMITER //
CREATE TRIGGER verifage
    BEFORE INSERT
    ON Réservations
    FOR EACH ROW
BEGIN
    DECLARE client_age INT;
    SELECT age INTO client_age FROM Clients WHERE id = NEW.client_id;

    IF client_age < 21 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'âge minimum pour louer une voiture est de 21 ans';
    END IF;
END;//
```
---


### 3. Trigger de Vérification de la Validité du Permis de Conduire lors de l'Ajout d'un Client

- **Nom du Trigger** : `verifvalidite`
- **Événement** : `BEFORE INSERT` sur la table `Clients`
- **Objectif** :
    - Valide le format et l'unicité du numéro de permis de conduire lors de l'ajout d'un client.

#### Code SQL :

```sql
DELIMITER //
CREATE TRIGGER verifvalidite
    BEFORE INSERT
    ON Clients
    FOR EACH ROW
BEGIN
    IF LENGTH(NEW.permis_conduire) != 15 OR NEW.permis_conduire NOT REGEXP '^[A-Z0-9]+$' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le permis de conduire doit être unique et composé de 15 caractères ';
    END IF;
END;//
```
---


### 4. Trigger de Vérification de la Disponibilité d'une Voiture avant Réservation

- **Nom du Trigger** : `verifdispo`
- **Événement** : `BEFORE INSERT` sur la table `Réservations`
- **Objectif** :
    - Vérifie que la voiture demandée est disponible avant de finaliser une réservation.

#### Code SQL :

```sql
DELIMITER //
CREATE TRIGGER verifdispo
    BEFORE INSERT
    ON Réservations
    FOR EACH ROW
BEGIN
    DECLARE car_available BOOLEAN;
    SELECT disponible INTO car_available FROM Voitures WHERE id = NEW.voiture_id;

    IF car_available = FALSE THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'La voiture sélectionnée nest pas disponible';
    END IF;
END;//
```
---

### 5. Trigger pour Éviter les Chevauchements de Réservations pour la Même Voiture

- **Nom du Trigger** : `evitechevauchement`
- **Événement** : `BEFORE INSERT` sur la table `Réservations`
- **Objectif** :
    - Empêche la création de réservations dont les dates se chevauchent pour une même voiture.

#### Code SQL :

```sql
DELIMITER //
CREATE TRIGGER evitechevauchement
    BEFORE INSERT
    ON Réservations
    FOR EACH ROW
BEGIN
    DECLARE overlap_count INT;
    SELECT COUNT(*) INTO overlap_count
    FROM Réservations
    WHERE voiture_id = NEW.voiture_id
      AND (NEW.date_debut BETWEEN date_debut AND date_fin
           OR NEW.date_fin BETWEEN date_debut AND date_fin
           OR date_debut BETWEEN NEW.date_debut AND NEW.date_fin);

    IF overlap_count > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Chevauchement de réservation pour cette voiture';
    END IF;
END;//
```