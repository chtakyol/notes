
This was the homework;
>1. Fastlane ve GitHub Actions Kullanımı: * Fastlane ve GitHub Actions gibi CI/CD araçlarını kullanarak bir mobil uygulamanın CI/CD sürecini oluşturmalı ve uygulamayı otomatik olarak dağıtım kanallarına gönderebilmelidir. +
>2. Farklı Environments ve Branch Yapıları: * Farklı ortamlar (environments) için dağıtım yapılmalıdır. Development ve master branch'leri üzerinden yapılandırmalar yapılmalı, her branch için uygun ayarlamalar gerçekleştirilmelidir. + 
>3. Uygulamanın Dağıtımı: * Uygulama en az iki ayrı versiyon halinde dağıtılmalıdır: 
>	1. Production versiyonu +
>	2. Test versiyonu  +
>4. Unit ve UI Test Entegrasyonu: * CI sürecinde Unit ve UI Testleri entegre edilmeli, test koşulları sağlanmadığı durumda pipeline fail olmalı

I'll divide 6 step;
 - [y] Building on your local
 - [y] Build in Github Actions
 - [y] Signing on local
 - [y] Signing on Github Actions
 - [y] Building with Fastlane and Deploying to Play Store on your local
 - [y] Building with Fastlane and Deploying to Play Store on Github Actions

Intro
### Building on your local
This is simple step you are always doing with play button :smile: but you can also build with from terminal using these command. However, you can also use terminal commands for more control.
```
./gradlew build
./gradlew clean build
```
### Building on Github Actions
We are using Github Action for remote, because it is free.
In repository we have `.github/workflow` directory. The `.yml` file in this directory is trigger Github Actions.
Here is example for just building on push any branch.
 ``` yml
 name: Build

on: push

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: set up JDK 1.8
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set Up JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: 17

      - name: Build with Gradle
        run: ./gradlew build
      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v3

      - name: Build Project
        run: ./gradlew assemble

```

### Signing on local environment
For signing I need create keystore.jks file. I've generated with `keytool` tool.

#### Create Keystore file
These `my-release-key.jks`, `my-key-alias` are user parameters. First time key create...
``` bash
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
```
This will ask you bunch of question like name of executer, company, location etc. And after that you will have `my-release-key.jks` . Here is `keystore name` and `alias` is important because we are going to use later.

After creating keystore file you can check the generated files **path** and size **with** following commands;
```bash
ls -l my-release-key.jks
du -h my-release-key.jks
```

Also in app level gradle need to know this keystore file and alias.
For this we create `signingConfigs` block.
And also there is `buildTypes` block, that we refer here later while signing.
``` gradle
signingConfigs {  
    create("release") {  
        storeFile = file("../my-release-key.jks")  
        storePassword = System.getenv("KEYSTORE_PASSWORD")  
        keyAlias = System.getenv("KEY_ALIAS")  
        keyPassword = System.getenv("KEY_PASSWORD")  
    }  
}
  
buildTypes {  
    getByName("release") {  
        isMinifyEnabled = false  
        proguardFiles(  
            getDefaultProguardFile("proguard-android-optimize.txt"),  
            "proguard-rules.pro"  
        )  
        signingConfig = signingConfigs.getByName("release")  
    }

    getByName("debug") {  
        signingConfig = signingConfigs.getByName("debug")  
    }  
}
```

We can add the all the information for creating signed release buuut, no it's internet so we need to encrypt those vales we'll talk this later. But adding those values to environment variables and we can access from there.
#### Add keyStorefile, keyStorePassword, keyAlias, keyPassword to environment variable
We can add these variables to envrinoment variable with `export` in your terminal.
```bash
export KEYSTORE_FILE="/path/to/my-release-key.jks" 
export KEYSTORE_PASSWORD="" 
export KEY_ALIAS="my-key-alias"
export KEY_PASSWORD=""
```

And you can check with echo command if they are added or not
```bash
echo $KEYSTORE_FILE
```

>**Note:** sometimes the `"` is became a tilted with autocomplate on and this cause terminals error in this case I had to restart session. And these variables are added for sessions, when you restart they will disappear.

After we've add those we can test locally with assemble command see if the keystore is working or not.
```bash
./gradlew assembleRelease
```

or 
```bash
./gradlew bundle
```


### Signing on Github Actions

We can sign and crate artifacts multiple ways, at the end Github Actions is basically a remote computer.
Here the target is running `./gradlew assembleRelease` or `./gradlew bundle` on Github Actions.
#### Adding environment variables to Github Actions secret
We need to access those important values via environment variable and we can manage this on Github Actions adding those values to `secrets`.
We need to add keystore on encrypted form.
##### Encrypt keystore file
I've use openssl for encrypting and decrypting my files. It's looks like this.
``` bash
openssl base64 -in my-release-key.jks -out my-release-key.jks.base64
```

After encrypting be sure out file is really generated :). You can check the out files size, existanc and location with following command.
```bash
ls -l ./my-release-key.jks.base64 -> exist?
realpath ./my-release-key.jks.base64 -> path
```

And after we can add this encrypted out files content to `secret`, open file copy content to value section, open file with `cat ./my-release-key.jks.base64`
##### Adding secrets
Here is how we can add these `secrets`;
- Repository settings 
- Secrets and Variables (Under Security block)
- Actions 
- New repository secret 
- Add key and value.

| KEY               | VALUE                                  |
| ----------------- | -------------------------------------- |
| KEYSTORE_FILE     | Content of ./my-release-key.jks.base64 |
| KEYSTORE_PASSWORD | Your password                          |
| KEY_ALIAS         | Your created alias `my-key-alias`      |
| KEY_PASSWORD      | Your password                          |

##### The build.yml file
After adding these secrets we need to decode the keystore file and also we have to add these values to environment variable so gradle can access. Code is very human I think.
```yml
name: Build  
  
on: push  
  
jobs:  
  build:  
    runs-on: ubuntu-latest  
    steps:  
      - name: Checkout  
        uses: actions/checkout@v4  
  
      - name: Set Up JDK  
        uses: actions/setup-java@v4  
        with:  
          distribution: 'zulu'  
          java-version: 17  
  
      - name: Setup Gradle  
        uses: gradle/actions/setup-gradle@v3  
  
      - name: Decode and Write Keystore File  
        run: |  
          echo "${{ secrets.KEYSTORE_FILE }}" | openssl base64 -d -out my-release-key.jks  
        env:  
          KEYSTORE_FILE: ${{ secrets.KEYSTORE_FILE }}  
  
      - name: Build the project  
        run: ./gradlew assembleRelease --no-daemon  
        env:  
          KEYSTORE_FILE: ${{ secrets.KEYSTORE_FILE }}  
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}  
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}  
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
```

##### For debuggin purpose I've use this blocks

For verifying if my release key is created or not.
```yml
      - name: Verify Keystore File  
        run: |  
          if [ -s my-release-key.jks ]; then  
          echo "Keystore file created successfully."  
          ls -l my-release-key.jks  
          else  
          echo "Keystore file is empty or missing!" && exit 1  
          fi  
```

Checking environment variable is really there and also checking path.
```yml
      - name: Debug Environment Variables  
        run: |  
          echo "KEYSTORE_PASSWORD is set"   
          echo "KEY_ALIAS is set"  
          echo "KEY_PASSWORD is set"  
        env:  
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}  
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}  
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}  
  
      - name: Check Keystore File Existence  
        run: ls -l ./my-release-key.jks  
  
      - name: Check Keystore Path  
        run: realpath ./my-release-key.jks  
```
### Building with Fastlane and Deploying to Play Store on your local
Fastlane is a automation tool that handle manuel build like `./gradle bundle` etc.

##### Initialization
First we need to installs this tool doc is pretty helpful for here. https://docs.fastlane.tools/getting-started/android/setup/ (homebrew works)

And in repository directory we have to init with `fastlane init` command the doc is also helpful here.

We also need to Google Service Account this also mentioned on docs. And it's pretty clear and up to date. So continue over fastlane doc.

After initialization we'll get fastlane directory and in this directory we'll have `Appfile` and `Fastfile`.
The `Appfile` include generic settings and paths.
The `Fastfile` is include built types and automation logic.

##### Building and signing

My Fastfile looks like this, again code is very human. Here the blocks named `internal_test`  in Fastfile, `fastlane internal_test` can be run on terminal.
And `upload_to_play_store` upload the to build. As you can see here we set `track: "internal`And `release_status: "draft"` because my app have not publish yet and I'm internally testing.
Also notice that we are also using environment variables, so before `fastlane internal_test` you should add those values.
```Ruby
default_platform(:android)  

lane :internal_test do  
  keystore_path = File.expand_path("../my-release-key.jks", __dir__)  
  gradle(  
    task: 'bundle',  
    build_type: 'Release',  
    properties: {  
      "android.injected.signing.store.file" => keystore_path,  
      "android.injected.signing.store.password" => ENV["KEYSTORE_PASSWORD"],  
      "android.injected.signing.key.alias" => ENV["KEY_ALIAS"],  
      "android.injected.signing.key.password" => ENV["KEY_PASSWORD"]  
    }  
  )  
  upload_to_play_store(  
      track: "internal",  
      release_status: 'draft',  
      skip_upload_apk: true,  
      skip_upload_metadata: true,  
      skip_upload_images: true,  
      skip_upload_screenshots: true  
  )  
end
```

> Note: There is also function for internal google play store upload named `upload_to_play_store_internal_app_sharing` and it's a lie. So don't waste time.
### Building with Fastlane and Deploying to Play Store on Github Actions
Next step is run this `fastlane` on Github Actions.
For this I'm using new `deploy.yml` file. Basically I set the environment variables and install `Fastlane`. 
We are also encrypting the Google Playstore Service authentication file.  And it is `secrets`.
```yml
name: Deploy to Play Store  
  
on:  
  push:  
    branches:  
      - master  
  
jobs:  
  deploy:  
    runs-on: ubuntu-latest  
  
    steps:  
      - name: Checkout Repository  
        uses: actions/checkout@v4  
  
      - name: Set Up Ruby  
        uses: ruby/setup-ruby@v1  
        with:  
          ruby-version: '3.1'  
  
      - name: Install Dependencies  
        run: |  
          gem install bundler  
          bundle install  
  
      - name: Install OpenSSL  
        run: sudo apt-get install -y openssl  
  
      - name: Decode and Write Keystore File  
        run: |  
          echo "${{ secrets.KEYSTORE_FILE }}" | openssl base64 -d -out my-release-key.jks  
        env:  
          KEYSTORE_FILE: ${{ secrets.KEYSTORE_FILE }}  
  
      - name: Decrypt Play Store Credentials File  
        run: |  
          echo "${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT }}" | openssl base64 -d -out ./fastlane/playstore_credentials.json  
  
      - name: Install Fastlane  
        run: sudo gem install fastlane  
  
      - name: Deploy to Internal Testing  
        run: fastlane internal_test  
        env:  
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}  
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}  
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
```






name: Build test apk  
  
on:  
  push:  
    branches:  
      - develop  
  
jobs:  
  build:  
    runs-on: ubuntu-latest  
    steps:  
      - name: Checkout  
        uses: actions/checkout@v4  
  
      - name: Set Up JDK  
        uses: actions/setup-java@v4  
        with:  
          distribution: 'zulu'  
          java-version: 17  
  
      - name: Setup Gradle  
        uses: gradle/actions/setup-gradle@v3  
  
      - name: Decode and Write Keystore File  
        run: |  
          echo "${{ secrets.KEYSTORE_FILE }}" | openssl base64 -d -out my-release-key.jks  
        env:  
          KEYSTORE_FILE: ${{ secrets.KEYSTORE_FILE }}  
  
      - name: Verify Keystore File  
        run: |  
          if [ -s my-release-key.jks ]; then  
          echo "Keystore file created successfully."  
          ls -l my-release-key.jks  
          else  
          echo "Keystore file is empty or missing!" && exit 1  
          fi  
  
      - name: Debug Environment Variables  
        run: |  
          echo "KEYSTORE_PASSWORD is set"   
          echo "KEY_ALIAS is set"  
          echo "KEY_PASSWORD is set"  
        env:  
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}  
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}  
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}  
  
      - name: Check Keystore File Existence  
        run: ls -l ./my-release-key.jks  
  
      - name: Check Keystore Path  
        run: realpath ./my-release-key.jks  
  
      - name: Build the project  
        run: ./gradlew clean assembleDebug  
  
      - name: Upload APK  
        uses: actions/upload-artifact@v3  
        with:  
          name: test-apk  
          path: app/build/outputs/apk/release/*.apk