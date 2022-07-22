# C# Live Project Code Summary


### Overview  
During the last two weeks of my Software Development Bootcamp at The Tech Academy I worked with several other students to build and develop an ASP.NET core MVC  web application in C#. We completed the minimum viable product (MVP) within a two week sprint and in addition to the MVP stories, I was able to complete several additional user stories ahead of time. The program was for a theater company and I was responsible for creating the section of the website that displayed and maintained the cast members' information.

### Technologies used:  
ASP.NET core MVC  
C#  
CSS  
Bootstrap  
Font-awesome  



### Highlights of the code I wrote during this sprint.
Basic CRUD Functionality/Photo Upload and Conversion  
Page Styling  
Search function  
Access Restriction  


##### Basic CRUD Functionality/Photo Upload and Conversion

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


###### Page Styling
   The page styling was adjusted based off of a color palette defined by the project-manager and some basic styling guidelines. A combination of bootstrap, basic CSS, and Razor syntax markup were used to manage the appearance of the page. For the Cast Member Index page the design requested different sections based off of the different productions that were currently being performed by the theater. Most of the code below is devoted to manipulating the content in that way.
   The biggest challenge was to populate the view with the photos from the database. This involved passing those data from the database as a byte[], converting it to a string which could be converted to an image by the src tag of the <img> element.
   Razor syntax code  
   [CSS code](#####-index-css-styling)  
   Next section  
	
Index Page gif of 

Generating different sections for different productions and populating photos to the view.

    CastMembers/Views/Index.cshtml

    @{
    // This creates an IEnumerable object which finds the number of distinct shows that are playing at the theater
    IEnumerable<string> showsEnum = Model.Select(x => x.ProductionTitle).Distinct();

    // This converts the above IEnumerable to a List
    List<string> showsList = showsEnum.ToList();

    // this starts a loop which effectively creates a different section on the index page for each production that is occuring at the theater
    for (int i = 0; i < showsList.Count(); i++)
    {
        // This prints the name of the show that is being represented in this section.
        <h4>@showsList[i]</h4>
        <hr class="cast_member-index--breakline" />
        <div class="cast_member-index--row">
            <!--This loop adds actors to this section of the page if the show that they are acting in is the same as the section (e.g. if they are acting in Les Mis and this is the Les Mis section they will be added)-->
            @foreach (var cast in Model)
            {
		if (cast.ProductionTitle == showsList[i])
		{
		    <div class="card cast_member-index--cards" style="width: 18rem;">
			@{
			    // This line of code takes the byte[] data from cast.Photo converts it to a base 64 string and puts it in a string variable
			    string imgString = Convert.ToBase64String(cast.Photo);
			    // This creates a string that will convert the base64string back into an image when it is included in the src tag of the image
			    string imgsrc = string.Format("data:image/png;base64,{0}", imgString);
			}
			<img class="card-img-bottom" src="@imgsrc" />
			<div class="cast_member-index--cardoverlay">
			// These blocks create font-awesome icons that can be used as links
			    @Html.ActionLink("‎",
					     "Edit",
					     "CastMembers",
					     new {id = cast.CastMemberId},
					     new { @class = "fa fa-pencil-square fa-4x cast_member-index--editoverlay" }
					     )
			    @Html.ActionLink("‎",
					     "Delete",
					     "CastMembers",
					     new { id = cast.CastMemberId },
					     new { @class = "fa fa-trash-o fa-4x cast_member-index--deleteoverlay" }
					     )
			</div>
			// This link is adjusted using CSS to cover the entire area of the image so that the whole image can be used as a link
			<a href="/Prod/CastMembers/Details/@cast.CastMemberId" class="cast_member-index--detailslink"></a>
			<div class="card-body cast_member-index--cardbody d-flex justify-content-center">
			    <h5 class="card-title">@cast.ProductionTitle</h5>
			</div>
		    </div>
		}
	    }
	</div>
	<br />
   	}
	
##### Index CSS Styling:

		/*=================================
	    Cast Member Index Page
	*/

	/* -breakline- */
	.cast_member-index--breakline {
	    color: white;
	    background-color: var(--main-color);
	    width: 100%;
	}

	/* -Add Member Button */
	.cast_member-index--addcastlink {
	    text-decoration: none;
	}

	    .cast_member-index--addcastlink:hover {
		text-decoration: none;
	    }

	.cast_member-index--createbutton {
	    background-color: var(--secondary-color);
	    font-weight: bold;
	}

	    .cast_member-index--createbutton:hover {
		background-color: var(--secondary-color--dark);
		font-weight: bold;
	    }

	/* -Page Title- */
	.cast_member-index--title {
	    text-align: center;
	    margin-top: 80px;
	    margin-left: auto;
	    margin-right: auto;
	}


	/*   ---Style cards/overlays---   */

	.cast_member-index--cards {
	    margin-right: 20px;
	}

	.cast_member-index--row {
	    display: flex;
	}

	.cast_member-index--cardoverlay {
	    position: absolute;
	    top: 0;
	    left: 0;
	    width: 100%;
	    height: 100%;
	    background: rgba(0, 0, 0, 0);
	    transition: background 0.5s ease;
	}

	.cast_member-index--cards:hover .cast_member-index--cardoverlay {
	    background: rgba(0, 0, 0, .3);
	}

	.cast_member-index--cardbody {
	    background-color: var(--dark-color);
	}

	/*   ---Edit, Delete, and Details Buttons---   */

	/* -Edit- */
	.cast_member-index--editoverlay {
	    position: absolute;
	    top: -1%;
	    left: 0;
	    background: rgba(0, 0, 0, 0);
	    transition: opacity 0.5s ease;
	    opacity: 0%;
	    text-decoration: none;
	    color: var(--secondary-color);
	    z-index: 11;
	}

	.cast_member-index--cards:hover .cast_member-index--editoverlay {
	    opacity: 100%;
	    cursor: pointer;
	    text-decoration: none;
	}

	.cast_member-index--editoverlay:hover {
	    color: var(--main-color);
	}

	/* -Delete- */
	.cast_member-index--deleteoverlay {
	    position: absolute;
	    top: 0;
	    right: 0;
	    background: rgba(0, 0, 0, 0);
	    transition: opacity 0.5s ease;
	    opacity: 0%;
	    color: var(--secondary-color);
	    z-index: 11;
	}

	.cast_member-index--cards:hover .cast_member-index--deleteoverlay {
	    opacity: 100%;
	    cursor: pointer;
	}

	.cast_member-index--deleteoverlay:hover {
	    color: var(--main-color);
	}


	/* -Details- */
	.cast_member-index--detailslink {
	    position: absolute;
	    top: 0;
	    left: 0;
	    width: 100%;
	    height: 100%;
	    z-index: 10;
	}


	/* -Search Bar Styling- */
	/*   ---Alter input focus color---   */
	.cast_memeber-index-inputglow {
	    background-color: #a6a6a6;
	    margin: 30px auto 10px;
	}

	.cast_memeber-index-inputglow:focus {
	    border-color: var(--secondary-color);
	    box-shadow: 0 0 10px var(--secondary-color);
	    background-color: var(--light-color);
	}

	.cast_member-index--positivebutton {
	    background-color: var(--secondary-color);
	    font-weight: bold;
	}

	.cast_member-index--positivebutton:hover {
	    background-color: var(--secondary-color--dark);
	}

	.cast_member-index--buttoncontainer{
	    text-align: center;
	}

	/*
	    END Cast Member Index Page
	==================================*/
	


##### Search Function
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



##### Access Restriction
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

