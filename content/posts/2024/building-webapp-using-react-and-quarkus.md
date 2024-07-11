---
title: "Building a WebApp using React and Quarkus"
date: 2024-07-05T23:21:14-04:00
draft: false
categories: 
    - Programming
tags: 
    - React
    - Quarkus
    - WebApp
---

My first website was an adventure straight out of 2014, built with Dreamweaver and some basic HTML and CSS. Picture a static site, lovingly crafted to display the research and papers from our lab. The end product was, well, let’s just say not exactly internet-breaking. But hey, it was my first foray into the wild world of web development!

Fast forward 10 years, after diving deep into data science, machine learning, and platform engineering, I realized something horrifying: I was still a web development noob. Cue the dramatic music. This simply would not do! So, armed with determination and copious amounts of coffee, I embarked on a quest to master web app development and fill in the gaping holes in my skill set.

After researching, I chose the following tech stack:

- **JavaScript library**: React + TypeScript
- **UI framework**: shadcn
- **State management**: Zustand
- **REST query**: react-query
- **CSS**: Tailwind CSS
- **Builder**: Vite
- **Backend**: Quarkus (a Java framework)

I highly recommend checking out Robin Wieruch's [blog](https://www.robinwieruch.de/react-libraries/) for insights on various recommended technologies. I chose Quarkus for the backend because it simplifies creating Docker images for deployment and fits well with SPAs, which I will detail later.

### Create the Project

Follow the [guide](https://ui.shadcn.com/docs/installation/vite) at shadcn to create the project.

You may see warning when importing sources using `@`. Add the below snippet to `compilerOptions` in the `tsconfig.app.json` as well.

```json
{
  "compilerOptions": {
    // ...
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "./src/*"
      ]
    }
    // ...
  }
}

```

### Navigating the Pits of Despair

#### Proxy Shenanigans with React and TypeScript

Imagine my frustration when trying to add a proxy to `package.json` or use `http-proxy-middleware`, only to be met with an unending series of CORS issues. My solution? Allow all CORS from the backend, since I’m hosting both frontend and backend together. For those not in the same boat, consider setting up a proxy server in React ([example](https://dev.to/codeofrelevancy/how-to-set-up-a-proxy-server-in-react-dealing-with-cors-87e)).
#### The Importance of Error Capturing

Using react-query to fetch data felt like upgrading from a tricycle to a Harley. It caches responses and refetches only when necessary. However, I made a rookie mistake: not capturing potential errors from processing the response.
```typescript
export const updateTodo = async (todo: Ttodo): Promise<void> => {
    const response = await fetch(`http://localhost:8080/1.0/todo/update`, {
        method: "PUT",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify(todo),
    });

    if (!response.ok) {
        throw new Error("Failed to update todo");
    } else {
        console.log(response);
    }
    return;
};
```

In the above code, `response.json()` threw an error when the response was empty, and I didn’t catch it. This oversight turned debugging into a treasure hunt. Lesson learned: always capture errors!
#### Keeping Query Keys Neat and Tidy

React-query’s `useMutation` hook allows cache invalidation based on `queryKey`. For example, the code below ensures all queries with `todos` at the first index in their `queryKey` are invalidated. Think of it as Marie Kondo-ing your query cache.

```typescript
const { mutate: updateTodoMutation } = useMutation({
    mutationFn: updateTodo,
    onSuccess: () => {
        console.log("updateTodoMutation success");
        queryClient.invalidateQueries({ queryKey: ['todos'] }).then(() => { return });
    },
    onMutate: () => {
        console.log("updateTodoMutation onMutate");
    },
    onError: (error) => {
        console.error("updateTodoMutation onError", error);
    },
    onSettled: () => {
        console.log("updateTodoMutation onSettled");
    },
});

```

#### Creating Submodules in Quarkus for a React Project

Hosting a SPA with Quarkus requires all contents to be under `/src/main/webui` ([Quarkus Quinoa](https://docs.quarkiverse.io/quarkus-quinoa/dev/index.html)), including `package.json`. Manually copying files is a chore, so creating a submodule saves the day. For instance, create separate GitHub repos like [ydeng11/Tai](https://github.com/ydeng11/Tai) and [ydeng11/Tai-UI](https://github.com/ydeng11/Tai-UI), then run `git submodule add https://github.com/ydeng11/Tai-UI.git ./src/main/webui` in your Quarkus project’s root directory. Voilà, everything works like magic!
### Conclusion

After spending four evenings coding this app and navigating these challenges, I can confidently say that frontend development has evolved leaps and bounds from when I built that static website a decade ago. My advice? Don’t sweat the basics of HTML and CSS too much. A crash course will suffice to get you started. Embrace tools like ChatGPT as your coding sidekick. You’ll learn heaps more through practice and hands-on experience.