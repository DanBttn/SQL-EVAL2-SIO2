Trigger n°1
---
Evenement : `AFTER INSERT` sur la table `reservation`
---
Objectif : Mettre à jour la disponibilité de la voiture à 0 lorsqu'une réservation est effectuée
---
#### Code SQL :

```sql
BEGIN

UPDATE Voitures
SET Voitures.disponible = 0
WHERE Voitures.id = new.Voitures.id;

END
```

Trigger n°2

Evenement : `BEFORE INSERT` sur la table `reservation`

Objectif : Vérifier l'âge du Client avant une Réservation

#### Code SQL :

```sql
BEGIN

SELECT Clients.age into @age
FROM Clients
WHERE Clients.id = new.client_id;

IF @age <21 THEN

SIGNAL SQLSTATE "45000"

Set MESSAGE_TEXT = "Votre âge doit être minimum à 21 ans";

END IF;

END

```

Trigger n°3
Evenement : `BEFORE INSERT` sur la table `clients`

Objectif : Vérifier la validité du permis de conduire avant la création d'un nouveau client

#### Code SQL :

```sql

BEGIN
    IF NEW.permis_conduire IS NULL THEN
 SIGNAL SQLSTATE '45000'
 SET MESSAGE_TEXT = 'Un numéro de permis de conduire valide est requis pour l''inscription.';
    END IF;

    IF LENGTH(NEW.permis_conduire) != 15 THEN
     SIGNAL SQLSTATE '45000'
     SET MESSAGE_TEXT = 'Le numéro de permis de conduire doit avoir exactement 15 caractères.';
    END IF;

    IF NEW.permis_conduire NOT REGEXP '^[A-Za-z0-9]{15}$' THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Le numéro de permis de conduire doit être alphanumérique et avoir 15 caractères.';
    END IF;

    IF EXISTS (SELECT 1 FROM Clients WHERE permis_conduire = NEW.permis_conduire) THEN
  SIGNAL SQLSTATE '45000'
  SET MESSAGE_TEXT = 'Ce numéro de permis de conduire est déjà utilisé par un autre client.';
    END IF;
END
```
Trigger n°4

Evenement : `BEFORE INSERT` sur la table `reservation`

Objectif : Vérifier la disponibilité de la voiture avant une réservation

#### Code SQL :

```sql
BEGIN

SELECT Voitures.disponible INTO @dispo
FROM Voitures
WHERE Voitures.id = new.voiture_id;

IF @dispo = 0 THEN
    SIGNAL SQLSTATE "45000"
    SET MESSAGE_TEXT = "La voiture demandée n'est pas disponible.";
END IF;

END
```



Trigger n°5

Evenement : `BEFORE INSERT` sur la table `reservation`

Objectif : Eviter les chevauchements de réservations sur la même voiture

#### Code SQL :

```sql
BEGIN
SELECT COUNT(*) INTO @chevauchement
FROM Reservations
WHERE voiture_id = new.voiture_id
  AND (date_debut BETWEEN new.date_debut AND new.date_fin
    OR date_fin BETWEEN new.date_debut AND new.date_fin);

IF @chevauchement > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'La voiture demandée est déjà réservée pour cette période.';
END IF;
END
```