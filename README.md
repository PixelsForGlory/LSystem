# L-System
Type based [Lindenmayer system](https://en.wikipedia.org/wiki/L-system).  This library generates l-systems that are based on collection of class types. 

[![Build status](https://ci.appveyor.com/api/projects/status/o3tifl663x9caedo/branch/master?svg=true)](https://ci.appveyor.com/project/LlamaBot/lsystem/branch/master)

## Building
Nothing special.  Build from solution.

## Usage

The examples below show a context sensitive l-system and a parametric l-system.  Instead of using the traditional string to represent the system a list of objects are used instead.

Below are classes and interfaces a developer will need to implement to create his/her own system:

- `ICopy` should be implemented by types that represent the state of an l-system during a production run.  When traversing down a supporting branch of a system, the state will need to be deep copied.
- `ILSystemModule` represents a module in the l-system.  In the context of the wikipedia article, this interface represents variables and constants in the examples.
- `ILSystemQueryableModule` represents a module in the l-system that can be queried during production.  In the context of the wikipedia article, this interface represents "Parametric grammars".  These are useful if trying to draw the l-system.
- `LSystemProduction` represents a context sensitive grammar for a production of the l-system.  Has built in condition and probablity which can be defined for each production.  

### Context Sensitive Example

    public class ContextSensitiveModuleA : ILSystemModule<int>
    {
        public int F { get; private set; }

        public ContextSensitiveModuleA(int f)
        {
            F = f;
        }

        public void ChangeState(LSystemState<int> systemState) { /* Does nothing */ }
    }

    public class ContextSensitiveModuleB : ILSystemModule<int>
    {
        public int F { get; private set; }

        public ContextSensitiveModuleB(int f)
        {
            F = f;
        }

        public void ChangeState(LSystemState<int> systemState) { /* Does nothing */ }
    }

    public class ContextSensitiveProduction1 : LSystemProduction<int>
    {
        public ContextSensitiveProduction1() : base(null, typeof(ContextSensitiveModuleA), null){}
        protected override bool Condition(LinkedListNode<LSystemNode<int>> currentNode)
        {
            return true;
        }

        protected override bool Probability(LinkedListNode<LSystemNode<int>> currentNode)
        {
            return LSystemTest.RandomNumberGenerator.NextDouble() <= 0.4f;
        }

        protected override List<LSystemNode<int>> CreateSuccessor(int stepNumber, LinkedListNode<LSystemNode<int>> currentNode)
        {
            var a = (ContextSensitiveModuleA)currentNode.Value.NodeModule;
            var node = new LSystemNode<int>(stepNumber, new ContextSensitiveModuleA(a.F + 1));
            return new List<LSystemNode<int>>(new[] { node });
        }
    }

    public class ContextSensitiveProduction2 : LSystemProduction<int>
    {
        public ContextSensitiveProduction2() : base(null, typeof(ContextSensitiveModuleA), null) { }

        protected override bool Condition(LinkedListNode<LSystemNode<int>> currentNode)
        {
            return true;
        }

        protected override bool Probability(LinkedListNode<LSystemNode<int>> currentNode)
        {
            return LSystemTest.RandomNumberGenerator.NextDouble() <= 0.6f;
        }

        protected override List<LSystemNode<int>> CreateSuccessor(int stepNumber, LinkedListNode<LSystemNode<int>> currentNode)
        {
            var a = (ContextSensitiveModuleA)currentNode.Value.NodeModule;
            var node = new LSystemNode<int>(stepNumber, new ContextSensitiveModuleB(a.F - 1));
            return new List<LSystemNode<int>>(new[] { node });
        }
    }

    public class ContextSensitiveProduction3 : LSystemProduction<int>
    {
        public ContextSensitiveProduction3() : base(typeof(ContextSensitiveModuleA), typeof(ContextSensitiveModuleB), typeof(ContextSensitiveModuleA)) { }

        protected override bool Condition(LinkedListNode<LSystemNode<int>> currentNode)
        {
            var b = (ContextSensitiveModuleB)currentNode.Value.NodeModule;
            return b.F < 4;
        }

        protected override bool Probability(LinkedListNode<LSystemNode<int>> currentNode)
        {
            return true;
        }

        protected override List<LSystemNode<int>> CreateSuccessor(int stepNumber, LinkedListNode<LSystemNode<int>> currentNode)
        {
            // ReSharper disable PossibleNullReferenceException
            var leftContext = (ContextSensitiveModuleA)currentNode.Previous.Value.NodeModule;
            var predecessor = (ContextSensitiveModuleB)currentNode.Value.NodeModule;
            var rightContext = (ContextSensitiveModuleA)currentNode.Next.Value.NodeModule;
            // ReSharper restore PossibleNullReferenceException

            var node = 
                new LSystemNode<int>(stepNumber, new ContextSensitiveModuleB(leftContext.F + rightContext.F),
                new LinkedList<LSystemNode<int>>(new[] { new LSystemNode<int>(stepNumber, new ContextSensitiveModuleA(predecessor.F)) }));

            return new List<LSystemNode<int>>(new[] { node });
        }
    }
    
    // In code somwhere...
    
     var axiom =
        new List<LSystemNode<int>>()
        {
            new LSystemNode<int>(0, new ContextSensitiveModuleA(1)),
            new LSystemNode<int>(0, new ContextSensitiveModuleB(2)),
            new LSystemNode<int>(0, new ContextSensitiveModuleA(3))
        };

        var productions =
        new List<LSystemProduction<int>>
        {
            new ContextSensitiveProduction1(),
            new ContextSensitiveProduction2(),
            new ContextSensitiveProduction3()
        };

    var contextSensitiveLSystem = new LSystem<int>(axiom, productions);

    contextSensitiveLSystem.RunProduction();
    contextSensitiveLSystem.GetCurrentDerivation()));
    
### Parametric Example
 
    public class ParametricState : ICopy<ParametricState>
    {
        public int PositionX;
        public int PositionY;
        public int Rotation;

        public override string ToString()
        {
            return string.Format("{0},{1},{2}", PositionX, PositionY, Rotation);
        }

        public ParametricState ShallowCopy()
        {
            throw new NotImplementedException();
        }

        public ParametricState DeepCopy()
        {
            var returnData = new ParametricState
            {
                PositionX = PositionX,
                PositionY = PositionY,
                Rotation = Rotation
            };

            return returnData;
        }
    }

    public class ParametricModuleEndPoint : ILSystemModule<ParametricState>
    {
        public void ChangeState(LSystemState<ParametricState> systemState) { /* Do nothing */ }
    }

    public class ParametricModuleMoveForward : ILSystemModule<ParametricState>
    {
        public int StepsForward { get; private set; }

        public ParametricModuleMoveForward(int stepsForward)
        {
            StepsForward = stepsForward;
        }

        public void ChangeState(LSystemState<ParametricState> systemState)
        {
            switch(systemState.CurrentState.Rotation)
            {
                case 0:
                    systemState.CurrentState.PositionY += StepsForward;
                    break;
                case 90:
                    systemState.CurrentState.PositionX += StepsForward;
                    break;
                case 180:
                    systemState.CurrentState.PositionY -= StepsForward;
                    break;
                case 270:
                    systemState.CurrentState.PositionX -= StepsForward;
                    break;
            }
        }
    }

    public class ParametricModuleRotate : ILSystemModule<ParametricState>
    {
        public int Rotation { get; private set; }

        public ParametricModuleRotate(int rotation)
        {
            Rotation = rotation;
        }

        public void ChangeState(LSystemState<ParametricState> systemState)
        {
            systemState.CurrentState.Rotation += Rotation;

            if(systemState.CurrentState.Rotation >= 360)
            {
                systemState.CurrentState.Rotation = 0;
            }
        }
    }

    public class ParametricModulePoint : ILSystemQueryableModule<ParametricState>
    {
        public int QueriedPositionX { get; private set; }
        public int QueriedPositionY { get; private set; }

        public void ChangeState(LSystemState<ParametricState> systemState) { /* Does nothing */ }

        public void QueryState(LSystemState<ParametricState> systemState)
        {
            QueriedPositionX = systemState.CurrentState.PositionX;
            QueriedPositionY = systemState.CurrentState.PositionY;
        }
    }

    public class ParametricProdction1 : LSystemProduction<ParametricState>
    {
        public ParametricProdction1() : base(null, typeof(ParametricModuleEndPoint), null){}

        protected override bool Condition(LinkedListNode<LSystemNode<ParametricState>> predecessor)
        {
            return true;
        }

        protected override bool Probability(LinkedListNode<LSystemNode<ParametricState>> predecessor)
        {
            return true;
        }

        protected override List<LSystemNode<ParametricState>> CreateSuccessor(int stepNumber, LinkedListNode<LSystemNode<ParametricState>> predecessor)
        {
            var forwardNode = new LSystemNode<ParametricState>(stepNumber, new ParametricModuleMoveForward(1));
            var pointNode = new LSystemNode<ParametricState>(stepNumber, new ParametricModulePoint());
            var rotationNode = new LSystemNode<ParametricState>(stepNumber, new ParametricModuleRotate(90));
            var endPointNode = new LSystemNode<ParametricState>(stepNumber, new ParametricModuleEndPoint());

            return new List<LSystemNode<ParametricState>>(new[] { forwardNode, pointNode, rotationNode, endPointNode });
        }
    }

    public class ParametricProduction2 : LSystemProduction<ParametricState>
    {
        public ParametricProduction2() : base(null, typeof(ParametricModuleMoveForward), null){}

        protected override bool Condition(LinkedListNode<LSystemNode<ParametricState>> predecessor)
        {
            return true;
        }

        protected override bool Probability(LinkedListNode<LSystemNode<ParametricState>> predecessor)
        {
            return true;
        }

        protected override List<LSystemNode<ParametricState>> CreateSuccessor(int stepNumber, LinkedListNode<LSystemNode<ParametricState>> predecessor)
        {
            var module = (ParametricModuleMoveForward) predecessor.Value.NodeModule;
            var forwardNode = new LSystemNode<ParametricState>(stepNumber, new ParametricModuleMoveForward(module.StepsForward + 1));
            
            return new List<LSystemNode<ParametricState>>(new[] { forwardNode });
        }
    }
    
    // Somewhere in code
     var axiom =
        new List<LSystemNode<ParametricState>>()
        {
            new LSystemNode<ParametricState>(0, new ParametricModuleEndPoint())
        };

        var productions =
            new List<LSystemProduction<ParametricState>>
            {
                new ParametricProdction1(),
                new ParametricProduction2()
            };

        var parametricLSystem = new LSystem<ParametricState>(axiom, productions);

        var intialState =
            new LSystemState<ParametricState>(
                new ParametricState()
                {
                    PositionX = 0,
                    PositionY = 0,
                    Rotation = 0
                });

        contextSensitiveLSystem.RunProduction();
        parametricLSystem.GetCurrentDerivation(intialState);
 
## Citations
Przemyslaw Prusinkiewicz, Mark James, and Radomír Měch. 1994. Synthetic topiary. In Proceedings of the 21st annual conference on Computer graphics and interactive techniques (SIGGRAPH '94). ACM, New York, NY, USA, 351-358. 
