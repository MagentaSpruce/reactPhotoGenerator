# reactPhotoGenerator
This React app uses the unsplash API to retrieve infinite images on scroll after user search input &amp; with clickable images.

This React project utilizes the UnSplash API to retrieve photos from: https://api.unsplash.com/photos/ &&
https://api.unsplash.com/search/photos/

A general walkthrough of the pertinent React code is provided below:

First the initial fetch images function is created.
```React
function App() {
  const fetchImages = async () => {
    let url;
    url = `${mainUrl}?clientID=9Y97As321RXVNaHvoyjP2kmhuZCINXN-SvbqEOJvh18`;
    try {
      const response = await fetch(url);
      const data = await response.json()
    } catch (error) {
      console.log(error);
      
    }
  };
```

Next the user and statevalues are set up.
```React
function App() {
  const [loading, setLoading] = useState(false);
  const [photos, setPhotos] = useState([]);

  const fetchImages = async () => {
    setLoading(true)
    let url;
    url = `${mainUrl}?clientID=9Y97As321RXVNaHvoyjP2kmhuZCINXN-SvbqEOJvh18`;
    try {
      const response = await fetch(url);
      const data = await response.json();
      console.log(data);
      
    } catch (error) {
      console.log(error);
      setLoading(false)
    }
  };

  useEffect(()=>{
    fetchImages()
  },[])
  return <h2>stock photos starter</h2>;
}
```

After the response is checked and an array of objects is successfully retreived then enviornmental variables are set-up to allow access to the UnSplash API without giving out access to the key.
```React
const clientID = `?client_id=${process.env.REACT_APP_PUBLIC_KEY}`;

    let url;
    url = `${mainUrl}${clientID}`;
```

Next the response and return are worked on to display images.
```React
      const response = await fetch(url);
      const data = await response.json();
      setPhotos(data)
      setLoading(false)

  return (
    <main>
      <section className="search">
        <form className="search-form">
          <input type="text" placeholder="search" className="form-input" />
          <button type='submit' className='submit-btn' onClick={handleSubmit}>
            <FaSearch/>
          </button>
        </form>
      </section>
    </main>
  );
```

Next the handleSubmit function is created.
```React
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("hello");
  };
```

Next the display photo part of the return is created.
```React
      <section className="photos">
        <div className="photos-center">
          {photos.map((image, index) => {
            return <Photo key={index} {...image} />;
          })}
        </div>
      </section>
```

Next loading is set up to listen for scroll events.
```React
      <section className="photos">
        <div className="photos-center">
          {photos.map((image, index) => {
            return <Photo key={index} {...image} />;
          })}
        </div>
        {loading && <h2 className="loading">Loading...</h2>}
      </section>
```

Next the photo component is worked on by destructuring data from the API object such as urls, alt_description, ect in order ti start displaying images.
```React
const Photo = ({ urls: { regular }, alt_description }) => {
  return (
    <article className="photo">
      <img src={regular} alt={alt_description} />
    </article>
  );
};
```

At this point images should be rendering to the screen. Next photo info is added in.
```React
const Photo = ({
  urls: { regular },
  alt_description,
  likes,
  user: {
    name,
    portfolio_url,
    profile_image: { medium },
  },
}) => {
  return (
    <article className="photo">
      <img src={regular} alt={alt_description} />
      <div className="photo-info">
        <div className="">
          <h4>{name}</h4>
          <p>{likes}</p>
        </div>
        <a href={portfolio_url}>
          <img src={medium} alt={name} className="user-img" />
        </a>
      </div>
    </article>
  );
};
```

Now that pictures with their information and links are rendering to the page the infinite scroll was worked on starting with an event listener for when the app loads. If innerHeight + scrollY >= body height then the next batch of data should be fetched and displayed.
```React
  useEffect(() => {
    const event = window.addEventListener("scroll", () => {
      // console.log(`innerHeight ${window.innerHeight}`);
      // console.log(`scrollY ${window.scrollY}`);
      // console.log(`body height ${document.body.scrollHeight - 5}`);
      if (!loading && window.innerHeight + window.scrollY >= document.body.scrollHeight) {
        console.log("it worked!");
      }
    });
    return () => window.removeEventListener("scroll", event);
  }, []);
```

Now logic is created to get new images once the bottom of the document has been reached by using a useEffect that updates whenever the page changes.
```React
  useEffect(() => {
    fetchImages();
  }, [page]);
```

Once the bottom of the document is reached the page is increased.
```React
      if (
        !loading &&
        window.innerHeight + window.scrollY >= document.body.scrollHeight - 4
      ) {
        setPage((oldPage) => {
          return oldPage + 1;
        });
      }
    });
```

Now instead of replacing images when a new event occurs the images are all added to the same array.
```React
    try {
      const response = await fetch(url);
      const data = await response.json();
      setPhotos((oldPhotos) => {
        return [...oldPhotos, ...data];
      });
      setLoading(false);
```

Now new images are being added once the end of the scroll event is reached (the page value changes). Next the search input is focused on starting with controlling the input component with a statevalue..
```React
  const [query, setQuery] = useState("");
  
          <form className="search-form">
          <input
            type="text"
            placeholder="search"
            className="form-input"
            value={query}
            onChange={(e) => {
              setQuery(e.target.value);
            }}
          />
```

Next the fetch url is changed dynamically to render user search results.
```React
    let url;
    const urlPage = `&page=${page}`;
    const urlQuery = `&query=${query}`;

    if (query) {
      url = `${searchUrl}${clientID}${urlPage}${urlQuery}`;
    } else {
      url = `${mainUrl}${clientID}${urlPage}`;
    }
```

Default is 10 pictures but search result generates the array inside of the total results object - so the results are in a different location.
```React
      setPhotos((oldPhotos) => {
        if (query) {
          return [...oldPhotos, ...data.results];
        } else {
          return [...oldPhotos, ...data];
        }
```

When new search is done, the old pictures need to be wiped away.
```React
      setPhotos((oldPhotos) => {
        if (query && page === 1) {
          return data.results;
        } else if (query) {
          return [...oldPhotos, ...data.results];
        } else {
          return [...oldPhotos, ...data];
        }
      });
```


***End walkthrough
