# JSON Path and YAML

## JSON Path - Dictionaries

```json
{
    "vehicles": {
        "car": {
            "color": "blue",
            "price": "$20,000"
        },
        "bus": {
            "color": "white",
            "price": "$120,000"
        },
    }
}
```

## JSON Path - Root Query

```json
$.vehicles.car
$.vehicles.bus.price
```

## JSON Path - List

```json
[
    "car",
    "bus",
    "truck",
    "bike"
]
```

```json
{
    "vehicles": {
        "car": {
            "color": "blue",
            "price": "$20,000"
            "wheels": [
                {
                "model": "X345ET",
                "localtion": "front-right"   
                },
                {
                "model": "X345GX",
                "localtion": "front-left"   
                },
                {
                "model": "X345MV",
                "localtion": "rear-right"   
                },
                {
                "model": "X245MV",
                "localtion": "rear-left"   
                }
            ]
        }
    }
}
```

```json
$.car.wheels[1].model
```

## JSON PATH - Query

Get all numbers greater than 40

```json
$[ Check if each item in the array > 40 ]
Check if => ? ()

$[ ?( each item in the list > 40 )]
each item in the list => @

$[ ?( @ > 40 )]
@ == 40 @ in [40,43,45]
@ != 40 @ nin [40,43,45]
```

```json
{
  "prizes": [
    {
      "year": "2018",
      "category": "physics",
      "overallMotivation": "\"for groundbreaking inventions in the field of laser physics\"",
      "laureates": [
        {
          "id": "960",
          "firstname": "Arthur",
          "surname": "Ashkin",
          "motivation": "\"for the optical tweezers and their application to biological systems\"",
          "share": "2"
        },
        {
          "id": "961",
          "firstname": "Gérard",
          "surname": "Mourou",
          "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
          "share": "4"
        },
        {
          "id": "962",
          "firstname": "Donna",
          "surname": "Strickland",
          "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
          "share": "4"
        }
      ]
    },
    {
      "year": "2018",
      "category": "chemistry",
      "laureates": [
        {
          "id": "963",
          "firstname": "Frances H.",
          "surname": "Arnold",
          "motivation": "\"for the directed evolution of enzymes\"",
          "share": "2"
        },
        {
          "id": "964",
          "firstname": "George P.",
          "surname": "Smith",
          "motivation": "\"for the phage display of peptides and antibodies\"",
          "share": "4"
        },
        {
          "id": "965",
          "firstname": "Sir Gregory P.",
          "surname": "Winter",
          "motivation": "\"for the phage display of peptides and antibodies\"",
          "share": "4"
        }
      ]
    },
    {
      "year": "2018",
      "category": "medicine",
      "laureates": [
        {
          "id": "958",
          "firstname": "James P.",
          "surname": "Allison",
          "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
          "share": "2"
        },
        {
          "id": "959",
          "firstname": "Tasuku",
          "surname": "Honjo",
          "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2018",
      "category": "peace",
      "laureates": [
        {
          "id": "966",
          "firstname": "Denis",
          "surname": "Mukwege",
          "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
          "share": "2"
        },
        {
          "id": "967",
          "firstname": "Nadia",
          "surname": "Murad",
          "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2018",
      "category": "economics",
      "laureates": [
        {
          "id": "968",
          "firstname": "William D.",
          "surname": "Nordhaus",
          "motivation": "\"for integrating climate change into long-run macroeconomic analysis\"",
          "share": "2"
        },
        {
          "id": "969",
          "firstname": "Paul M.",
          "surname": "Romer",
          "motivation": "\"for integrating technological innovations into long-run macroeconomic analysis\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2014",
      "category": "peace",
      "laureates": [
        {
          "id": "913",
          "firstname": "Kailash",
          "surname": "Satyarthi",
          "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
          "share": "2"
        },
        {
          "id": "914",
          "firstname": "Malala",
          "surname": "Yousafzai",
          "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2017",
      "category": "physics",
      "laureates": [
        {
          "id": "941",
          "firstname": "Rainer",
          "surname": "Weiss",
          "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
          "share": "2"
        },
        {
          "id": "942",
          "firstname": "Barry C.",
          "surname": "Barish",
          "motivation": "\"for decisive co$.prizes[?(@.year == 2014)].laureates[*].firstnamentributions to the LIGO detector and the observation of gravitational waves\"",
          "share": "4"
        }
      ]
    },
    {
      "year": "2017",
      "category": "chemistry",
      "laureates": [
        {
          "id": "944",
          "firstname": "Jacques",
          "surname": "Dubochet",
          "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
          "share": "3"
        },
        {
          "id": "945",
          "firstname": "Joachim",
          "surname": "Frank",
          "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
          "share": "3"
        },
        {
          "id": "946",
          "firstname": "Richard",
          "surname": "Henderson",
          "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
          "share": "3"
        }
      ]
    },
    {
      "year": "2017",
      "category": "medicine",
      "laureates": [
        {
          "id": "938",
          "firstname": "Jeffrey C.",
          "surname": "Hall",
          "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
          "share": "3"
        },
        {
          "id": "939",
          "firstname": "Michael",
          "surname": "Rosbash",
          "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
          "share": "3"
        },
        {
          "id": "940",
          "firstname": "Michael W.",
          "surname": "Young",
          "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
          "share": "3"
        }
      ]
    }
  ]
}
```

* Give this a shot. Try to find Malala in the below list of Noble Prize Winners.

```json
$.prizes[5].laureates[1]
```
* Retreive the `first and 4th element` in the list

```json
[
  "car",
  "bus",
  "truck",
  "bike"
]
```

```json
$.[0,3]
```

## JSON PATH - Wildcard


```json
{
    "vehicles": {
        "car": {
            "color": "blue",
            "price": "$20,000"
        },
        "bus": {
            "color": "white",
            "price": "$120,000"
        },
    }
}
```

```bash
# Get car's color
$.cat.color

# Get bus's color

# Get all colors
$.*.color

# Get all prices
$.*.price
```


```json
{
    "vehicles": {
        "car": {
            "color": "blue",
            "price": "$20,000"
            "wheels": [
                {
                "model": "X345ET",
                "localtion": "front-right"   
                },
                {
                "model": "X345GX",
                "localtion": "front-left"   
                },
                {
                "model": "X345MV",
                "localtion": "rear-right"   
                },
                {
                "model": "X245MV",
                "localtion": "rear-left"   
                }
            ]
        }
    }
}
```

```json
# Get all wheels
$[*].wheels[*].model
```

```json
{
  "car": {
    "color": "blue",
    "price": "$20,000"
  },
  "bus": {
    "color": "white",
    "price": "$120,000"
  }
}
```

```json
$.[*].color
[
  "blue",
  "white"
]
```

```json
[
  {
    "model": "KDJ39848T",
    "location": "front-right"
  },
  {
    "model": "MDJ39485DK",
    "location": "front-left"
  },
  {
    "model": "KCMDD3435K",
    "location": "rear-right"
  },
  {
    "model": "JJDH34234KK",
    "location": "rear-left"
  }
]
```

```json
{
  "employee": {
    "name": "john",
    "gender": "male",
    "age": 24,
    "address": {
      "city": "edison",
      "state": "new jersey",
      "country": "united states"
    },
    "payslips": [
      {
        "month": "june",
        "amount": 1400
      },
      {
        "month": "july",
        "amount": 2400
      },
      {
        "month": "august",
        "amount": 3400
      }
    ]
  }
}
```

```json
$.employee.payslips[*].amount
[
  1400,
  2400,
  3400
]
```


```json
{
    "prizes:" [
        ...
    ]
}
```

```json
$.prizes[*].laureates[*].firstname
[
  "Arthur",
  "Gérard",
  "Donna",
  "Frances H.",
  "George P.",
  "Sir Gregory P.",
  "James P.",
  "Tasuku",
  "Denis",
  "Nadia",
  "William D.",
  "Paul M.",
  "Kailash",
  "Malala",
  "Rainer",
  "Barry C.",
  "Kip S.",
  "Jacques",
  "Joachim",
  "Richard",
  "Jeffrey C.",
  "Michael",
  "Michael W."
]
```

Let's mix things up. Try to find the `first names of all winners of year 2014` in the below list of Noble Prize Winners.

```json
{
    "prizes:" [
        ...
    ]
}
```

```json
$.prizes[?(@.year == 2014)].laureates[*].firstname
```

## JSON PATH - Lists


```json
#$[START:END]
$[0:8]
```

```json
#$[START:STEP:END]
$[0:8]
```

```json
#$[LAST 1 ELEMENT]
$[:-1]
```

```json
#$[LAST 3 ELEMENT]
$[:-3]
```


## Use case Kubernetes

1. Identify the `kubectl` commnand

2. Familiarize with `JSON` output

3. Form the `JSON PATH` query

```bash
.item[0].spec.containers.[0].image
```

4. Use the `JSON PATH` query with `kubectl` command
   
```bash
$ kubectl get pods -o=jsonpath='{ .items[0].spec.containers[0].image}'
```

5. `JSON PATH` Examples

```bash
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'
```

```bash
kubectl get nodes -o=jsonpath='{ .items[*].status.nodeInfo.architecture}'
```

```bash
kubectl get nodes -o=jsonpath='{ .items[*].status.capacity.cpu}'
```

```bash
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{.items[*].status.capacity.cpu}'
```

```bash
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}'
```

```bash 
# {FOR EACH NODE}
#   {PRINT NODE NAME}{TAB}{CPU NODES}{NEW LINE}
# {END FOR EARCH NODE}
kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
```

```bash 
kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
```

```bash
kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
```

```bash
kubectl get nodes --sort-by=.metadata.name
```

```bash
kubectl get nodes --sort-by=.status.capacity.cpu
```

```bash
oc get is -n openshift -o=custom-columns=IMAGE:.metadata.name,TAGS:.status.tags[*].tag
```

```bash
oc get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu,MEM:.status.capacity.memory
```

```
# Get all nodes in json format
kubectl get nodes -o json > /opt/outputs/nodes.json

# Get node01 in json format
kubectl get nodes node01 -o json > /opt/outputs/node01.json

# Get node names
kubectl get nodes -o jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt

# Get node os
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt

# Get users
kubectl config view --kubeconfig=/root/my-kube-config -o jsonpath='{.users[*].name}' > /opt/outputs/users.txt

# Get pv sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt

# Get pv sorted by capacity and only the columns NAME and CAPACITY
kubectl get pv -o=custom-columns='NAME:.metadata.name,CAPACITY:.spec.capacity.storage' --sort-by=.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt

# Get only the context en json format for "aws-user"
kubectl config view --kubeconfig=/root/my-kube-config -o jsonpath='{.contexts[?(@.context.user=="aws-user")]}' > /opt/outputs/aws-context-name
```