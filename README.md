
# 📱 Guide de Déploiement iOS avec Codemagic (React Native)

Ce document explique étape par étape comment déployer une application React Native sur iOS en utilisant Codemagic depuis un environnement Windows.

## ⚙️ Prérequis

- **Code source hébergé sur GitHub**  
  L’application doit être hébergée sur GitHub, de préférence dans l'organisation ou le compte de **LRC Group**.

- **Accès Codemagic**  
  Vous devez vous connecter à [Codemagic](https://codemagic.io/start/) via le compte GitHub de LRC Group.

- **Compte Apple Developer**  
  Assurez-vous d’avoir accès au [Compte Apple Developer](https://developer.apple.com/account/).  
  Créez un nouvel identifiant d’application (App ID) correspondant à l’identifiant unique de du projet (`com.lrcgroup.dtmoney`) présent dans le code source.

## 🧰 Étapes de configuration

### 1. Générer le projet iOS en local

Sur Windows, vous pouvez générer les fichiers nécessaires au build iOS sans exécuter le projet localement en utilisant :

```bash
npx react-native eject
npx react-native init DTMoney
```

> ⚠️ Vous ne pourrez pas lancer l'app iOS localement sous Windows. Ce build sert uniquement à préparer les fichiers de configuration.

### 2. 📄 Ajouter le fichier `codemagic.yaml`

Ajoutez un fichier `codemagic.yaml` à la racine du projet contenant la configuration du pipeline de build. Exemple minimal :

```yaml
workflows:
  ios-release:
    name: iOS Release Build
    environment:
      vars:
        XCODE_WORKSPACE: "ios/DTMoney.xcworkspace"
        XCODE_SCHEME: "VotreApp"
        BUNDLE_ID: "com.lrcgroup.dtmoney"
      ios_signing:
        distribution_type: app_store
        bundle_identifier: $BUNDLE_ID
        upload_bitcode: true
        automatic_code_signing: true
    scripts:
      - name: Install dependencies
        script: |
          npm install
      - name: Bundle iOS app
        script: |
          cd ios
          pod install
      - name: Build IPA
        script: |
          xcodebuild -workspace $XCODE_WORKSPACE -scheme $XCODE_SCHEME -configuration Release -sdk iphoneos archive -archivePath $HOME/build/DTMoney.xcarchive
          xcodebuild -exportArchive -archivePath $HOME/build/DTMoney.xcarchive -exportOptionsPlist ios/exportOptions.plist -exportPath $HOME/build/export
    artifacts:
      - build/export/*.ipa
```

### 3. 🔑 Configurer la signature iOS sur Codemagic

- Accédez à **App settings > Code signing** sur Codemagic.
- Ajoutez les fichiers suivants :
  - **Certificate (.p12)** : généré depuis votre compte Apple Developer.
  - **Provisioning Profile** : lié à l'identifiant de l'app.
- Codemagic peut aussi gérer la signature automatique si vous activez `automatic_code_signing: true`.

### 4. Déclencher le build

- Poussez vos changements sur la branche GitHub principale (par exemple `main` ou `release`).
- Allez sur Codemagic, choisissez le workflow et lancez le build.

## Personnalisation avec scripts

Vous pouvez ajouter des scripts personnalisés dans `codemagic.yaml` :

```yaml
    scripts:
      - name: Nettoyage
        script: |
          rm -rf node_modules
          npm install
      - name: Lint
        script: npm run lint
```

## 📦 Publication sur TestFlight

Une fois l'application compilée avec succès sur Codemagic, il est possible de la distribuer via TestFlight pour des tests internes ou externes. Voici les étapes à suivre :

### 1. Ajout de l'application sur App Store Connect

- Connectez-vous à [App Store Connect](https://appstoreconnect.apple.com/).
- Allez dans **Mes Apps** et cliquez sur le bouton "+" pour ajouter une nouvelle application.
- Renseignez les informations suivantes :
  - **Nom de l’app**
  - **Langue principale**
  - **Identifiant du bundle** (doit correspondre à celui du projet dans `app.json`)
  - **Plateforme** : iOS
  - **SKU** : identifiant unique (ex : `com.lrcgroup.monapp`)
- Cliquez sur **Créer**.

### 2. Vérification des profils d'approvisionnement et certificats

- Assurez-vous que le `Bundle Identifier` utilisé dans votre projet est bien associé à un **App ID** dans le [Apple Developer Portal](https://developer.apple.com/account/).
- Vérifiez que les **certificats de distribution** et les **profils d’approvisionnement** sont bien configurés sur Codemagic :
  - `Certificates > Upload certificate`
  - `Provisioning profiles > Upload .mobileprovision`

### 3. Envoi du build depuis Codemagic

- Dans Codemagic, après avoir configuré le fichier `codemagic.yaml`, déclenchez un build pour iOS.
- À la fin du build, vous verrez dans les logs la mention :
`Successfully uploaded .ipa to App Store Connect`
- Cela signifie que l'IPA a bien été transférée vers App Store Connect.

### 4. Configuration de TestFlight

- Retournez sur App Store Connect, section **TestFlight**.
- Vous verrez votre build apparaître après un petit délai (5 à 20 min).
- Cliquez sur le build, puis :
- Renseignez les informations nécessaires (notes de version, détails techniques).
- Cliquez sur **“Soumettre pour révision”** pour les tests externes.
- Pour un test **interne**, ajoutez les testeurs dans **Testeurs internes** et cliquez sur **Activer le test**.

### 5. Bonnes pratiques

- Pour éviter les rejets, vérifiez :
- Que toutes les autorisations demandées par l'app sont justifiées dans `Info.plist`
- Que le `build number` et la `version` sont mis à jour à chaque build
- Que les captures d'écran sont prêtes pour la soumission finale (même si pas nécessaires pour TestFlight)

---

## Résultat

À la fin du build, vous obtiendrez un fichier `.ipa` que vous pouvez soumettre via App Store Connect.

---

**Auteur :** Kaiser Styve  
**Dernière mise à jour :** 10 Juillet 2025
