# IBindingTypeConverter

## chatlogs

    ghuntley [11:48 AM] 
    @paulcbetts: can I get an explanation as to what ``GetAffinityForObjects`` is and how it works for the documentation? https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/BindingTypeConverters.cs#L91
    
    ghuntley [11:49 AM]
    https://github.com/ghuntley/ReactiveUI/blob/readthedocs/docs/sources/binding/type-converters.md
    
        
    paulcbetts [11:50 AM] 
    Return 0 if you can't convert an object and return like 50 if you can
    
    ghuntley [11:53 AM] 
    What happens if you return other than 0 (false)? Like 25 instead of 50?
    
    paulcbetts [11:54 AM] 
    Don't worry about that :)
    
    ghuntley [11:54 AM] 
    100 seems to be your favourite number https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/BindingTypeConverters.cs#L18
    
    ghuntley [12:10 PM] 
    thus my interpretation is
    
    ghuntley [12:10 PM]
    public int GetAffinityForObjects(Type fromType, Type toType)
           {
               if (fromType is string)
               {
                   return 100;
               }
               return 0;
           }
    
    paulcbetts [12:11 PM] 
    `fromType` is always `Type`
    
    ghuntley [12:16 PM] 
    thus (if fromtype = typeof(System.String))

    paulcbetts [12:21 PM] 
    :+1:


## Example - WIP

    using System;
    using ReactiveUI;
    using Conditions;
    using Splat;
    
    namespace MyCoolApp.Core.Converters
    {
        public class InverseStringIsNullEmptyOrWhitespaceToBoolTypeConverter : IBindingTypeConverter, IEnableLogger
        {
            public int GetAffinityForObjects(Type fromType, Type toType)
            {
                if (fromType == typeof(string))
                {
                    return 100; // any number other than 0 signifies conversion is possible.
                }
                return 0;
            }
    
            public bool TryConvert(object from, Type toType, object conversionHint, out object result)
            {
                Condition.Requires(from).IsNotNull();
                Condition.Requires(toType).IsNotNull();
                
                try
                {
                    result = !String.IsNullOrWhiteSpace(from);
                }
                catch (Exception ex)
                {
                    this.Log().WarnException("Couldn't convert object to type: " + toType, ex);
                    result = null;
                    return false;
                }
                
                return true;
            }
    
        }
    }

## Reference Material
* https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/PropertyBinding.cs#L50
* https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/Xaml/BindingTypeConverters.cs#L24

* http://stackoverflow.com/questions/23592231/how-do-i-register-an-ibindingtypeconverter-in-reactiveui
* https://github.com/reactiveui/ReactiveUI/commit/7fba662c7308db60cfda9e6fb3331b7cb514f14c

## Registration

    Locator.CurrentMutable.RegisterConstant(
        new MyCoolTypeConverter(), typeof(IBindingTypeConverter));

## Usage/Binding
[redacted]: you never need to worry about specifying the converter in the binding. the GetAffinityForObjects is there for bindings to determine their priority when receiving certain types.
[redacted]: let me see if i can understand what paul's magic rules for the result is - i'm pretty sure it's 0 - don't care, positive number - i can resolve this, and larger number wins
[redacted]: eureka https://github.com/reactiveui/ReactiveUI/blob/238524a922aed50f8141a1d26ff24b8f2b101b60/ReactiveUI/RegisterableInterfaces.cs#L155-L165
[@ghuntley]: ah that explains some things as to why the binding converter was firing on each view model load even when the binding was not wired onto a view binding.
[@ghuntley]: let’s say however theres a InverseStringIsWhitespaceEmpyOrNullToBoolConverter and a StringIsWhitespaceEmptyToNullToBoolConverter. ie. String.IsNullEmptyOrWhitespace(x) and !String.IsNullEmptyOrWhitespace(x) both registered into splat. How can I be specific as to which one should be used? Like both will return a compatible affinity but it will be the wrong one depending on the UI case use.