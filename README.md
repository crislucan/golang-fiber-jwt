Updated resetpasword code as such:

3.) In main.go add:
    router.Post("/resetpassword", controllers.ForgetPassword)
    
2.) In models/user.model.go add:
    ...
     23 type ForgetPasswordInput struct {
     24         Email           string `json:"email" validate:"required"`
     25         NewPassword     string `json:"password" validate:"required,min=8"`
     26         ConfirmPassword string `json:"passwordConfirm" validate:"required,min=8"`
     27 }
     ...
     
 3.) In controllers/auth.controller.go add:
 
    ...
      57 func ForgetPassword(c *fiber.Ctx) error {
 58         var payload *models.ForgetPasswordInput
 59 
 60         if err := c.BodyParser(&payload); err != nil {
 61                 return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"status": "fail", "message": err.Error()})
 62         }
 63 
 64         errors := models.ValidateStruct(payload)
 65         if errors != nil {
 66                 return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"status": "fail", "errors": errors})
 67         }
 68 
 69         var user models.User
 70         result := initializers.DB.First(&user, "email = ?", strings.ToLower(payload.Email))
 71         if result.Error != nil {
 72                 return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"status": "fail", "message": "User not found"})
 73         }
74 
 75         if payload.NewPassword != payload.ConfirmPassword {
 76                 return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"status": "fail", "message": "Passwords do not match"})
 77         }
 78 
 79         hashedPassword, err := bcrypt.GenerateFromPassword([]byte(payload.NewPassword), bcrypt.DefaultCost)
 80         if err != nil {
 81                 return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"status": "fail", "message": err.Error()})
 82         }
 83 
 84         // Update the user's password in the existing record
 85         user.Password = string(hashedPassword)
 86         result = initializers.DB.Save(&user)
 87         if result.Error != nil {
 88                 return c.Status(fiber.StatusBadGateway).JSON(fiber.Map{"status": "error", "message": "Failed to update password"})
 89         }
 90 
 91         return c.Status(fiber.StatusOK).JSON(fiber.Map{"status": "success", "message": "Password reset successful"})
 92 }
 
  ...
  
  
  In the call endpoint: http://<your-url>/api/auth/resetpassword
  pass in Body the json: {
    "email": "newadmin2@admin1.com",
    "password": "newpassword12345",
    "passwordConfirm": "newpassword12345"
}
    

# Golang, GORM, & Fiber: JWT Authentication

In this comprehensive guide, you'll learn how to implement JWT (JSON Web Token) authentication in a Golang application using GORM and the Fiber web framework. The REST API will be powered by a high-performance Fiber HTTP server, offering endpoints dedicated to secure user authentication, and persist data in a PostgreSQL database.

![Golang, GORM, & Fiber: JWT Authentication](https://codevoweb.com/wp-content/uploads/2023/01/Golang-GORM-Fiber-JWT-Authentication.webp)

## Topics Covered

- Run the Golang & Fiber JWT Auth Project
- Setup the Golang Project
- Setup PostgreSQL and pgAdmin with Docker
- Create the GORM Model
- Database Migration with GORM
    - Load the Environment Variables with Viper
    - Create the Database Pool with GORM
    - Migrate the GORM Model to the Database
- Create the JWT Authentication Controllers
    - SignUp User Fiber Context Handler
    - SignIn User Fiber Context Handler
    - Logout User Fiber Context Handler
- Get the Authenticated User
- Create the JWT Middleware Guard
- Register the Routes and Add CORS
- Testing the JWT Authentication Flow
    - Register a New Account
    - Log into the Account
    - Access Protected Routes
    - Logout from the API


Read the entire article here: [https://codevoweb.com/golang-gorm-amp-fiber-jwt-authentication/](https://codevoweb.com/golang-gorm-amp-fiber-jwt-authentication/)

