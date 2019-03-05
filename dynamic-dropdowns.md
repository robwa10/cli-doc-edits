### Dynamic Dropdowns

Sometimes, API endpoints require clients to specify a parent object in order to create or access the child resources. Imagine having to specify a company id in order to get a list of employees for that company. Since people don't speak in auto-incremented ID's, it is necessary that Zapier offer a simple way to select that parent using human readable handles.

Our solution is to present users a dropdown that is populated by making a live API call to fetch a list of parent objects. We call these special dropdowns "dynamic dropdowns."

To define one, you can provide the `dynamic` property on your field to specify the trigger that should be used to populate the options for the dropdown. The value for the property is a dot-separated concatenation of a trigger's key, the field to use for the value, and the field to use for the label.

```js
const App = {
  //...
  resources: {
    project: {
      key: 'project',
      //...
      list: {
        //...
        operation: {
          perform: () => {
            return [{ id: 123, name: 'Project 1' }];
          } // called for project_id dropdown
        }
      }
    },
    issue: {
      key: 'issue',
      //...
      create: {
        //...
        operation: {
          inputFields: [
            {
              key: 'project_id',
              required: true,
              label: 'Project',
              dynamic: 'projectList.id.name'
            }, // calls project.list
            {
              key: 'title',
              required: true,
              label: 'Title',
              helpText: 'What is the name of the issue?'
            }
          ]
        }
      }
    }
  }
};
```

In the UI, users will see something like this:

![screenshot of dynamic dropdown in Zap Editor](https://cdn.zapier.com/storage/photos/dd31fa761e0cf9d0abc9b50438f95210.png)

In the above code example the `dynamic` property makes reference to a resource. If you need to use a trigger to power a dynamic dropdown only make reference to the trigger's key.

```js
//...
issue: {
  key: 'issue',
  //...
  create: {
    //...
    operation: {
      inputFields: [
        {
          key: 'project_id',
          required: true,
          label: 'Project',
          dynamic: 'project.id.name'
        }, // will call the perform method of the trigger with the key: 'project'
        {
          key: 'title',
          required: true,
          label: 'Title',
          helpText: 'What is the name of the issue?'
        }
      ]
    }
  }
}
//...
```

In some cases you will need to power a dynamic dropdown but do not want to make the Trigger available to the end user. Here it is best practice to create the trigger and set `hidden: true` on it's `display` object.  

```js
const App = {
  //...
  triggers: {
    new_project: {
      key: 'project',
      noun: 'Project',
      // `display` controls the presentation in the Zapier Editor
      display: {
        label: 'New Project',
        description: 'Triggers when a new project is added.',
        hidden: true
      },
      operation: {
        perform: projectListRequest
      }
    },
    another_trigger: {
      // Another trigger definition...
    }
  }
};
```

Dynamic dropdown's within the same Trigger or Action can make use of the value set in a previous dynamic dropdown via `bundle.inputData`.

```js
const App = {
  //...
  triggers: {
    new_project: {
      key: 'project',
      noun: 'Project',
      display: {
        label: 'New Project',
        description: 'Triggers when a new project is added.',
      },
      operation: {
        // called for project_id dropdown
        perform: () => {
          return [{ id: 123, name: 'Project 1' }];
        }
      }
    },
    new_assignee: {
      key: 'assignee',
      noun: 'Assignee',
      // `display` controls the presentation in the Zapier Editor
      display: {
        label: 'New Assignee',
        description: 'Triggers when a new assignee is added to a project.',
        hidden: true
      },
      operation: {
        // called for assignee_id dropdown
        perform: () => {
          return z.request('http://example.com/api/v2/projects.json', {
            params: {
              project_id: bundle.inputData.project_id
            }
          })
          .then(response => z.JSON.parse(response.content));
          // assuming the response is [{ id: 123, name: 'Jane Doe' }]
      }
    }
  },
    issue: {
      key: 'issue',
      //...
      create: {
        //...
        operation: {
          inputFields: [
            {
              key: 'project_id',
              required: true,
              label: 'Project',
              dynamic: 'project.id.name'
            },
            {
              key: 'assignee_id',
              label: 'Assignee',
              dynamic: 'assignee.id.name',
              helpText: 'Who would you like to assignee the project to?'
            }
          ]
        }
      }
    }
  }
};
```

If you want your trigger to perform specific scripting for a dynamic dropdown then use `bundle.meta.prefill`. This can be useful if your dropdown needs to make use of [pagination](#whats-the-deal-with-pagination-when-is-it-used-and-how-does-it-work) to load more options.  

```js
const App = {
  //...
  resources: {
    project: {
      key: 'project',
      //...
      list: {
        //...
        operation: {
          canPaginate: true,
          perform: () => {
            if (bundle.meta.prefill) {
              // perform pagination request here
            } else {
              return [{ id: 123, name: 'Project 1' }];
            }
          }
        }
      }
    },
    issue: {
      key: 'issue',
      //...
      create: {
        //...
        operation: {
          inputFields: [
            {
              key: 'project_id',
              required: true,
              label: 'Project',
              dynamic: 'projectList.id.name'
            }, // calls project.list
            {
              key: 'title',
              required: true,
              label: 'Title',
              helpText: 'What is the name of the issue?'
            }
          ]
        }
      }
    }
  }
};
```
