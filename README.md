# C# Live Project Code Summary


###Overview
During the last two weeks of my Software Development Bootcamp at The Tech Academy I worked with several other students to build and develop an ASP.NET core MVC  web application in C#. We completed the minimum viable product (MVP) within a two week sprint and in addition to the MVP stories, I was able to complete several additional user stories ahead of time. The program was for a theater company and I was responsible for creating the section of the website that displayed and maintained the cast members' information.

###Technologies used:
ASP.NET core MVC
C#
CSS
Bootstrap
Font-awesome



Here are the highlights of the code I wrote during this sprint.

Basic CRUD Functionality/Photo Upload and Conversion
Page Styling
Search function
Access Restriction


###Basic CRUD Functionality/Photo Upload and Conversion
After the creation of the models I needed for the Cast Member section of the website, ASP.NET MVC provided the majority of the scaffolding necessary for CRUD functionality. However with an additional user story requesting the ability to upload pictures for all cast members, more functionality was required. This was achieved by having a file upload <input> tag added to the HTML/Razor form. To retrieve this datum I also had to include the use of the HttpPostedFileBase to return the data that was uploaded back to the controller.
One the file was in the controller it was simple to convert the photo file to a byte array (byte[]) which could then be stored in the database. in order for this process to run smoothly I needed to assure that the user was only uploading image files. To manage that, I added a try/catch block that would return a warning message to the user if they attempted to upload a file type other than an image.

        CastMembersController.cs
        
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "CastMemberId,Name,YearJoined,MainRole,Bio,CurrentMember,Character,CastYearLeft,DebutYear,ProductionTitle")] CastMember castMember, HttpPostedFileBase photoUpload)
        {
            try
            {
                if (photoUpload.ContentLength > 0)
                {
                    // Takes the HttpPostedFileBase object photoUpload and converts it to an Image object.
                    Image sourceImage = Image.FromStream(photoUpload.InputStream);

                    // Sets the castMember.Photo property to the byte array created with ConvertImage()
                    castMember.Photo = ConvertImage(sourceImage);
                }
            }
            // This catch block assures the user only uploads photo files, and provides them with feedback if they upload the wrong file type.
            catch (ArgumentException)
            {
                ViewBag.errorMessage = "You must enter an image file. (.png, .jpeg, .jpg etc.)";
                return View();
            }


            if (ModelState.IsValid)
            {
                db.CastMembers.Add(castMember);
                db.SaveChanges();
                return RedirectToAction("Index");
            }

            return View(castMember);
        }


###Page Styling
The page styling was adjusted based off of a color palette defined by the project-manager and some basic styling guidelines. A combination of bootstrap, basic CSS, and Razor syntax markup were used to manage the appearance of the page. 
	The biggest challenge was to populate the view with the photos from the database. This involved passing those data from the database as a byte[] converting it to a string that could then be converted to an image by the src tag of the <img> element.
Index Page gif of 


Code here:


###Search Function
Creating a search function was a fun and interesting task. For this example I created a new view that would represent the search results but would appear the same as the main index page. The controller collected all of the cast member information from the database, and I then filtered that, based on the search criteria, into a new list that was passed to the view.

    CastMembersController.cs

    // GET: Prod/CastMembers/Search
    public ActionResult Search(string searchTerm)
    {
        List<CastMember> allCastMembers = db.CastMembers.ToList();

        var resultsList = new List<CastMember>();

        foreach (var cast in allCastMembers)
        {
            if (cast.Name.ToLower().Contains(searchTerm.ToLower()) || cast.Bio.ToLower().Contains(searchTerm.ToLower()))
            {
                resultsList.Add(cast);
            }
        }

        return View(resultsList);
    }

search gif:



###Access Restriction
My final addition to the project was to adjust the website so that it could only be accessed by an admin user. MVC again made this fairly simple, and I was able to generate a default admin user on application startup that could be used to access the Create, Update, and Delete functionality, while disallowing access to these functions to any regular or anonymous user. 

    Startup.cs
    
    public partial class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            ConfigureAuth(app);
            CreateRolesAndUsers();
        }


        // This method is creating a default User role and admin user for logging in.
        private void CreateRolesAndUsers()
        {
            ApplicationDbContext context = new ApplicationDbContext();

            // In the same way that the line above creates a context to the database in general the following lines create a context which allows us to add users and roles
            var RoleManager = new RoleManager<IdentityRole>(new RoleStore<IdentityRole>(context));
            var UserManager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(context));


            //This checks to see if the Admin role exists and if it does not it creates that admin role on startup
            if (!RoleManager.RoleExists("Admin"))
            {

                //These lines create a new role and calls it "Admin"
                var role = new IdentityRole();
                role.Name = "Admin";
                RoleManager.Create(role);


                //Here we create an Admin superuser
                var user = new ApplicationUser();
                user.UserName = "Ransom";
                user.Email = "rdcadorette@gmail.com";

                string userPWD = "C#L1vePr0ject";

                //Uses UserManager object to add a user with the given credentials and password
                var chkUser = UserManager.Create(user, userPWD);


                // This adds a default User to the "Admin" Role
                if (chkUser.Succeeded)
                {
                    var result1 = UserManager.AddToRole(user.Id, "Admin");
                }
            }

        }
    }

