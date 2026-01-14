# Seting up the project

You will work on a project that demonstrates how to use JPA relationships in a Spring Boot application. More specifically how to use the `@ManyToOne`, and `@OneToMany` annotations. But first, you need to set up the project.

## Step 1: Fork the repository
Fork this repository [link](https://github.com/ek-osnb/spring-jpa-relations).

This will create a copy of the repository in your own GitHub account.

## Step 2: Navigating to your destination folder
Open up a terminal (Windows: Git Bash, Mac/Linux: Terminal), and navigate to the folder where you want to clone the repository to. You can use the `cd` command to change directories. For example:
```bash
cd Documents/third-semester
```

## Step 2: Clone the repository to your local machine
When your are in the destination folder, clone the forked GitHub repository to your local machine using the following command:
```bash
git clone <your-forked-repo-url>
```
Replace `<your-forked-repo-url>` with the URL of your forked repository, which you can find on your GitHub repo. Remember to use the SSH URL!


## Step 3: Open the project in IntelliJ IDEA
Open IntelliJ IDEA, and select `Open` from the welcome screen (or `File` -> `Open` if you have a project open). Navigate to the folder where you cloned the repository, select the project folder, and click `Open`.

## Step 4: Test that the project compiles and runs
To ensure that everything is set up correctly, try to run the application.

## Step 5: Explore the project structure
Take a look at the project structure and familiarize yourself with the different packages and classes.

**NOTE:** I have added extra interfaces to help you get familiar with the concept.

**Interfaces** help you define a contract that classes can implement. They are useful for defining common behaviors without dictating how those behaviors should be implemented. The service layer exposes interfaces, while the controller layer only depend on these interfaces, not the concrete implementations. This promotes loose coupling and makes it easier to swap out implementations if needed.