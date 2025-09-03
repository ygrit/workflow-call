🧑‍💻 TP GitHub Actions — Appel de Workflow (30–45 min)
🎯 Objectifs pédagogiques
Comprendre et utiliser les workflows réutilisables (workflow_call).
Savoir passer des paramètres d’un workflow à un autre.
Afficher dynamiquement des messages dans les logs GitHub Actions.
Savoir structurer un projet avec plusieurs workflows.
Mettre en place une logique conditionnelle simple dans un job.
⏱️ Durée estimée
30 à 45 minutes, en fonction du rythme et des extensions réalisées.

🛠️ Étapes du TP
1. Créer un dépôt GitHub (si ce n’est pas déjà fait)
Tu peux travailler sur un dépôt existant ou en créer un nouveau.

2. Créer un workflow enfant réutilisable
Dans le dossier .github/workflows/child.yml, créer un fichier contenant :

name: Child Workflow

on:
  workflow_call:
    inputs:
      name:
        required: true
        type: string
      time:
        required: false
        type: string
        default: "day"

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - name: Display greeting
        run: |
          if [ "${{ inputs.time }}" = "morning" ]; then
            echo "Good morning, ${{ inputs.name }}!"
          elif [ "${{ inputs.time }}" = "evening" ]; then
            echo "Good evening, ${{ inputs.name }}!"
          else
            echo "Hello, ${{ inputs.name }}!"
          fi
✅ Ce qu’on apprend ici :

Un workflow workflow_call
Utilisation de plusieurs inputs
Logique conditionnelle dans un run
3. Créer un workflow parent
Dans .github/workflows/parent.yml, créer un fichier contenant :

name: Parent Workflow

on:
  workflow_dispatch: # déclenchable manuellement

jobs:
  call-child-morning:
    uses: ./.github/workflows/child.yml
    with:
      name: "Alice"
      time: "morning"

  call-child-evening:
    uses: ./.github/workflows/child.yml
    with:
      name: "Bob"
      time: "evening"
✅ Ce qu’on apprend ici :

Utiliser workflow_dispatch (déclenchement manuel)
Appeler deux fois un même workflow avec des paramètres différents
Réutiliser un workflow dans plusieurs jobs
4. Lancer le TP
Pousse les deux fichiers dans ton repo.
Va dans Actions > Parent Workflow > Run workflow.
Observe les logs :

Tu devrais voir deux jobs (call-child-morning et call-child-evening).
Chacun affiche une salutation différente.
🧠 Bonus (si temps disponible)
🔁 Challenge 1 : demander les inputs à l’utilisateur
Remplace les appels dans parent.yml par un seul, avec des inputs dynamiques :

on:
  workflow_dispatch:
    inputs:
      user_name:
        description: "Nom de l'utilisateur"
        required: true
        default: "Charlie"
      moment:
        description: "Moment de la journée (morning, evening, day)"
        required: false
        default: "day"

jobs:
  call-child:
    uses: ./.github/workflows/child.yml
    with:
      name: ${{ inputs.user_name }}
      time: ${{ inputs.moment }}
Tu peux alors lancer le workflow et personnaliser les inputs à chaque run.

🧼 Challenge 2 : ajouter un job de nettoyage à la fin (factice)
Dans le parent.yml, ajoute un job final :

  cleanup:
    runs-on: ubuntu-latest
    needs: call-child
    steps:
      - run: echo "Cleaning up after the greeting..."
Parfait ! On peut ajouter une étape supplémentaire dans le TP pour montrer comment récupérer et utiliser un output d’un workflow enfant. Voici comment tu peux intégrer ça :

5. Bonus — Tester un output depuis le workflow enfant
1️⃣ Ajouter un output dans le workflow enfant (child.yml)
On va modifier le job greet pour qu’il renvoie le message de salutation comme output :

jobs:
  greet:
    runs-on: ubuntu-latest
    outputs:
      greeting-message: ${{ steps.greeting.outputs.message }}
    steps:
      - name: Set greeting message
        id: greeting
        run: |
          if [ "${{ inputs.time }}" = "morning" ]; then
            echo "message=Good morning, ${{ inputs.name }}!" >> $GITHUB_OUTPUT
          elif [ "${{ inputs.time }}" = "evening" ]; then
            echo "message=Good evening, ${{ inputs.name }}!" >> $GITHUB_OUTPUT
          else
            echo "message=Hello, ${{ inputs.name }}!" >> $GITHUB_OUTPUT

      - name: Display greeting
        run: echo "${{ steps.greeting.outputs.message }}"
✅ Ce qu’on apprend ici :

Définir un output d’un job (outputs + steps.id.outputs)
Propager des valeurs vers le workflow parent
2️⃣ Récupérer l’output dans le workflow parent (parent.yml)
jobs:
  call-child:
    uses: ./.github/workflows/child.yml
    with:
      name: "Dana"
      time: "morning"

  display-output:
    runs-on: ubuntu-latest
    needs: call-child
    steps:
      - run: echo "Le workflow enfant a renvoyé : ${{ needs.call-child.outputs.greeting-message }}"
✅ Ce qu’on apprend ici :

Attendre qu’un job enfant soit terminé (needs)
Récupérer et afficher un output dans le parent
3️⃣ Test pratique
Pousser les modifications dans ton repo.
Lancer le workflow parent.
Observer le job display-output qui montre le message provenant du workflow enfant.
